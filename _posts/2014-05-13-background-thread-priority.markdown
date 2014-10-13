---
layout: post
title: background thread priority 
date: 2014-05-13 11:34
published: false
---

Hello all, I got some trouble with thread priority. I have three thread, the main thread does less compute however I want it running with high priority. The other two threads, one is communicating with our app, and one is communicating with the serial port, do more IO work, and I want them running with relatively low priority. Because I need the two threads stop immediately when I remove the usb dongle, or start to run immediately when I insert the usb dongle.

At the following code, I use these three statements to set the main thread's priority:

      struct sched_param param;
      param.sched_priority = sched_get_priority_max(SCHED_FIFO);
      pthread_setschedparam(pthread_self(), SCHED_FIFO, &param);

and, for simplicity, I use `sleep(1)` in the two low priority thread to simulate the IO work.

This program is running on openwrt, `uname -a` is:

    Linux localhost 2.6.36.4brcmarm #7 SMP PREEMPT Fri Mar 21 10:04:49 CST 2014 armv7l GNU/Linux

and following is my source code:

    #define DONGLE_FILE             "/dev/ttyUSB0"
    #define DONGLE_NAME             "ttyUSB0"

    pthread_t thread_serial;
    pthread_t thread_app_server;

    pthread_attr_t attr_serial;
    pthread_attr_t attr_app_server;

    pthread_mutex_t mutex_serial = PTHREAD_MUTEX_INITIALIZER;
    pthread_mutex_t mutex_app_server = PTHREAD_MUTEX_INITIALIZER;

    void *serial_peer_run(void *ptr) {
      while(1) {
        log_info_msg("waiting for lock: serial peer\n");
        pthread_mutex_lock(&mutex_serial);
        log_info_msg("has got the lock: serial peer\n");

        sleep(1);

        log_info_msg("release the lock: serial peer\n");
        pthread_mutex_unlock(&mutex_serial);
      }

      return NULL;
    }

    void *app_server_run(void *ptr) {
      while(1) {
        log_info_msg("waiting for lock: app server\n");
        pthread_mutex_lock(&mutex_app_server);
        log_info_msg("has got the lock: app server\n");

        sleep(1);

        log_info_msg("release the lock: app server\n");
        pthread_mutex_unlock(&mutex_app_server);
      }

      return NULL;
    }

    static int nl_sock_ = 0;
    static struct sockaddr_nl nl_name_;
    static fd_set rset;

    int main(int argc, char *argv[]) {
      ///
      // set main thread schedule policy to FIFO
      //
      struct sched_param param;
      param.sched_priority = sched_get_priority_max(SCHED_FIFO);
      pthread_setschedparam(pthread_self(), SCHED_FIFO, &param);

      pthread_attr_init(&attr_serial);
      pthread_attr_init(&attr_app_server);
    
      //pthread_attr_setschedpolicy(&attr_app_server, SCHED_IDLE);

      pthread_create(&thread_serial, NULL, serial_peer_run, NULL);
      pthread_create(&thread_app_server, &attr_app_server, app_server_run, NULL);

      nl_name_.nl_family = AF_NETLINK;
      nl_name_.nl_pid = getpid();
      nl_name_.nl_groups = 1;

      nl_sock_ = socket(PF_NETLINK, SOCK_DGRAM, NETLINK_KOBJECT_UEVENT);
      if(-1 == nl_sock_) {
        fprintf(stderr, strerror(errno));
        return 1;
      }

      if(-1 == bind(nl_sock_, (struct sockaddr *)&nl_name_, sizeof(struct sockaddr_nl))) {
        fprintf(stderr, strerror(errno));
        return 1;
      }

      FD_ZERO(&rset);
      FD_SET(nl_sock_, &rset);

      while(1) {
        if(select(nl_sock_ + 1, &rset, NULL, NULL,  NULL) > 0) {
          /**
           * because rset only contains nl_sock_,  so if  select() returns positive value, it must be nl_sock_ can be read.
           */
          char buffer[512] = {0};
          recv(nl_sock_, buffer, sizeof(buffer), 0);

          /**
           * when usb dongle inserted, buffer is filled with:
           * add@/devices/pci0000:00/0000:00:0b.0/usb2/2-2/2-2:1.0/ttyUSB0
           */
          if(! strncmp(buffer, "add", 3) && strstr(buffer, DONGLE_NAME) && strstr(buffer, "devices")) {
            log_debug_msg("dongle " DONGLE_NAME " inserted.\n");

            sleep(2);
            log_trace_msg("release the lock: close serial\n");
            pthread_mutex_unlock(&mutex_serial);

            log_trace_msg("release the lock: app server\n");
            pthread_mutex_unlock(&mutex_app_server);
          }

          /**
           * when usb dongle is removed, buffer is filled with:
           * remove@/devices/pci0000:00/0000:00:0b.0/usb2/2-2/2-2:1.0/ttyUSB0
           */
          if(! strncmp(buffer, "remove", 6) && strstr(buffer, DONGLE_NAME) && strstr(buffer, "devices")) {
            log_debug_msg("dongle " DONGLE_NAME " removed.\n");

            log_debug_msg("waiting the lock of serial and app server\n");
            pthread_mutex_lock(&mutex_serial);
            pthread_mutex_lock(&mutex_app_server);
            log_debug_msg("has locked the serial port and app server\n");
          }
        }
      }

      pthread_join(thread_serial, NULL);
      pthread_join(thread_app_server, NULL);

      pthread_attr_destroy(&attr_serial);
      pthread_attr_destroy(&attr_app_server);

      close(nl_sock_);

      return 0;
    }

when this program runs on foreground, it's ok, the main thread got the high priority, when I remove the usb dongle, main thread will get the two mutexes immediately, and the printed log is:

    # ./multi-thread-test

    ....
    ....
    waiting the lock of serial and app server
    release the lock: app server
    waiting for lock: app server
    has got the lock: app server
    release the lock: serial peer
    waiting for lock: serial peer
    release the lock: app server
    waiting for lock: app server
    has locked the serial port and app server
    ...
    ...

but when this program runs with `nohup`, the main thread will not get the two mutexes immediately, it seems like that this two statements do no influence with the main thread's priority:

      param.sched_priority = sched_get_priority_max(SCHED_FIFO);
      pthread_setschedparam(pthread_self(), SCHED_FIFO, &param);

and the log is:

      # nohup ./multi-thread-test

        ....
        ....
        waiting the lock of serial and app server
        release the lock: app server
        waiting for lock: app server
        has got the lock: app server
        release the lock: serial peer
        waiting for lock: serial peer
        has got the lock: serial peer
        release the lock: serial peer
        release the lock: app server
        waiting for lock: app server
        has got the lock: app server
        waiting for lock: serial peer
        has got the lock: serial peer
        release the lock: serial peer
        release the lock: app server
        ...
        ...
        waiting for lock: app server
        has got the lock: app server
        release the lock: app server
        waiting for lock: serial peer
        has got the lock: serial peer
        release the lock: serial peer

I don't know why, any help will be appreciated.

Thanks!
