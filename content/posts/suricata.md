---
title: "Suricata Quickstart"
date: "2021-05-27"
description: "suricata"
categories:
  - Network Security
tags:
  - suricata
  - network security
---

## 1. How to run suricata

Just a `suricata` command with the default configuration should be enough for most caess, to see what the default configurations are, use `suricata --build-info`, the most important configurations in my macbook are:

```
Generic build parameters:
  Installation prefix:                     /usr/local/Cellar/suricata/6.0.2
  Configuration directory:                 /usr/local/etc/suricata/
  Log directory:                           /usr/local/var/log/suricata/

  --prefix                                 /usr/local/Cellar/suricata/6.0.2
  --sysconfdir                             /usr/local/etc
  --localstatedir                          /usr/local/var
  --datarootdir                            /usr/local/Cellar/suricata/6.0.2/share
```

So we know if we don't specify the config path and the output log path, suricata will read configuration from `/usr/local/etc/suricata/` and outputs log to `/usr/local/var/log/suricata`.

`suricata --dump-config` is alaso helpful.

### 1.1. Self building suricata

1. suricata honors the `autogen.sh`, `configure`, `make` build steps.
2. suricata relies on libhtp during building, we need to put the libhtp's source under suricata source root, then `./configure` will find it automatically
3. suricata relies on rust after 4.1, need to [install rust](https://www.rust-lang.org/tools/install) before `./configure`

### 1.2 Forefront topics

1. [Capture full session on alert](https://redmine.openinfosecfoundation.org/issues/120)

## 2. Rule management

### 2.1. Source listing

It's convenient to use the `suricata-update` command to manage rules, the same type of rules are usually grouped into a same source. The OISF fundation maintains an index of the famous sources here: [https://www.openinfosecfoundation.org/rules/index.yaml]()[^1], you can use `suricata-update list-sources` command to read the famous sources from this url, these famous sources can be considered as the builtin sources. The output of `suricata-update list-sources` looks like:

[^1]: This url is obtained from suricata-update's source code, the code path is /usr/local/lib/python3.9/site-packages/suricata/update/sources.py on my machine

```
Name: et/open
  Vendor: Proofpoint
  Summary: Emerging Threats Open Ruleset
  License: MIT
Name: et/pro
  Vendor: Proofpoint
  Summary: Emerging Threats Pro Ruleset
  License: Commercial
  Replaces: et/open
  Parameters: secret-code
  Subscription: https://www.proofpoint.com/us/threat-insight/et-pro-ruleset
Name: oisf/trafficid
  Vendor: OISF
  Summary: Suricata Traffic ID ruleset
  License: MIT
Name: ptresearch/attackdetection
  Vendor: Positive Technologies
  Summary: Positive Technologies Attack Detection Team ruleset
  License: Custom
```

### 2.2. Your own sources

Besides the builtin ones, you can use `suricata-update add-source` to add your own sources, if you look into the content of the https://www.openinfosecfoundation.org/rules/index.yaml, each source provides another url which contains the full rules in it, so adding your own source is just simple as putting your rules into a source file and expose that file on internet. Similarly, you can use `suricata-update remove-source` to remove an existing source.

### 2.3. Enable the sources you are interested

Once you are fine with your sources, you need an extra step to "enable" the sources(What a nuisance!) use the command `suricata-update enable-source`, note that the `et/open` source is always enabled by default, so you don't need to enable it explicitly.

### 2.4. Fetching the rules

Okay, now you can run `suricata-update update` to fetch the latest rules of all sources enabled - as sources would update their rules periodically, note again this will only fetch rules from the sources you have enabled using `suricata-update enable-source`(`et/open` excluded, as said).

Once `suricata-update udpate` finishes, all of the rules are write into a single file: /var/lib/suricata/rules/suricata.rules(this can be changed in suricata configuration file).

### 2.5. Writing your own rules and the rule SID allocation

SID here means "signature id'. If you need to add your own rules, please refer to https://doc.emergingthreats.net/bin/view/Main/SidAllocation to choose your SID properly.

## 3. Source code

### 3.1. Threading

**Concepts**:

Thread vars, Thread modules and Thread module slots:

In suricata's source code, `ThreadVars` represents a system thread, it also has a thread module slots which is a linked list of `TmModule`, on the other side, `ThreadVars` is wrapped by `Thread`, then all of the `Thread`s are included in `Threads`, meanwhile, `ThreadVars`s are connected to each other in a linked list too.

**Code flow**:

```
SuricataMain() --> RunModeDispatch()
    --> mode->RunModeFunc()
        --> Creates `ThreadVars`
            --> Gets Thread entry function(also called tm slot entry) and store it at `tm_func()` of `ThreadVars`,
                generally thread entry function is TmThreadsSlotPktAcqLoop()(src/tm-threads.c)
        --> Sets up slots and populates the required thread modules to the slots in sequence, slot's init/run/AcqPktLook
            functions are set to thread modules' relating functions
            --> pcap live mode, the following thread modules are appended into the slots sequencially 
                * ReceivePcap
                * DecodePcap
                * FlowWorker
                * RespondReject
        --> Spawn the thread with the `tm_func()` of the `ThreadVars` as the thread entry function
```

### 3.2. Packet pipeline

When a new packet arrives, it flows such a pipeline:

```
pcap callback --> the ReceivePcap slot --> the DecodePcap slot --> FlowWorker slot
                                                                        --> Detection engine
                                                                        --> Log Outputs
```

### 3.3. Detection engine and rules

If enabled, the detection engine is called in the FlowWorker slot, just have a look at the `Detect()` call in src/flow-worker.c:FlowWorker().

* DetectEngineThreadCtx

### 3.4. Outputs Log

**Concepts:**

* Output modules(aka log modules) response for the underlying log writing, an output module is represented by an `OutputModule` structure, defined in src/output.h
* `OutputModule` are classified by what callback function is set, see src/runmodes.c:SetupOutput(), e.g.
  * Flow logger
  * Stats logger
  * Packet logger
  * Transaction logger
  * etc...
* Each type of the output modules is defined in src/output-\*.h file, each has a global private `list` variable which is the head pointing to all output modules of this type.
* Each of the output modules types has a management structure called Root Logger structure, see src/output.c:OutputRegisterRootLoggers().

**Code flow:**

```
SuricataMain()
    --> PostConfLoadedSetup()
        --> RegisterAllModules()
            --> TmModuleLoggerRegister()
                --> OutputRegisterRootLoggers()
                    --> append the root loggers into the global list `registered_loggers`
                --> OutputRegisterLoggers()
                    --> register all of the output modules and append them into a global list `output_modules`
    --> PreRunPostPrivsDropInit()
        --> RunModeInitializeOutputs()
            --> Get and parse the "outputs" section from suricata.yaml
            --> According to the configurations in suricata.yaml, register every enabled output modules to the
                corresponding type of root logger(flow, packet, stats, etc...), during this step, the global private
                `list` of each root loggers is populated
            --> OutputSetupActiveLoggers()
                --> Since output modules may not be enabled in suricata.yaml, so after the above process,
                    some root loggers could end up with no output modules attached
                --> So this function is called to find out only the "active" root loggers
                --> OutputRegisterActiveLogger()
                    --> traverse the global list `registered_loggers` and find only the ones has output modules
                        attached, linked them into a new list: `active_loggers`
```

During logging step of the packet pipeline, the `active_loggers` is traversed hence all of the output modules of all types are called.

### 3.5. Packet arriving time

suricata relies on pcap to get the packet from network interface, when getting the packet, pcap will [populate the timestamp the packet arrives at the interface](https://github.com/the-tcpdump-group/libpcap/blob/fbcc461fbc2bd3b98de401cc04e6a4a10614e99f/pcap-netfilter-linux.c#L252), suricata gets this info and exposes it to us as well.

### 3.6. Bypass

Firstly, "bypass" here means skipping the further steps[^2] during processing a packet, instead of skipping further rules, which is what I thought, misunderstoodly.

[^2]: "Suricata reads a packet, decodes it, checks it in the flow table...", refer to https://www.stamus-networks.com/blog/2016/09/28/suricata-bypass-feature

Bypassing is implemented in two ways in suricata, local and capture, local means the bypassing logic exists in suricata's source code. Capture means the logic is provided by the various "captures"(AF_PACKET, NFQ, etc...) in linux kernel, to utilize it, suricata just call a function provided by kernel and declare the rule it want to bypass.

### 3.7. More developer resources[^3][^4]

[^3]: Developer wiki: https://redmine.openinfosecfoundation.org/projects/suricata/wiki
[^4]: Roadmap: https://redmine.openinfosecfoundation.org/projects/suricata/roadmap

## 4. Other resources

Suricata vs Snort: https://core.ac.uk/download/pdf/36699186.pdf
