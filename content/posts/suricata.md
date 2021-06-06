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

## How to run suricata

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

## Rule management

### source listing

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

### your own sources

Besides the builtin ones, you can use `suricata-update add-source` to add your own sources, if you look into the content of the https://www.openinfosecfoundation.org/rules/index.yaml, each source provides another url which contains the full rules in it, so adding your own source is just simple as putting your rules into a source file and expose that file on internet. Similarly, you can use `suricata-update remove-source` to remove an existing source.

### enable the sources you are interested

Once you are fine with your sources, you need an extra step to "enable" the sources(What a nuisance!) use the command `suricata-update enable-source`, note that the `et/open` source is always enabled by default, so you don't need to enable it explicitly.

### fetching the rules

Okay, now you can run `suricata-update update` to fetch the latest rules of all sources enabled - as sources would update their rules periodically, note again this will only fetch rules from the sources you have enabled using `suricata-update enable-source`(`et/open` excluded, as said).

Once `suricata-update udpate` finishes, all of the rules are write into a single file: /var/lib/suricata/rules/suricata.rules(this can be changed in suricata configuration file).

### Writing your own rules and the rule SID allocation

SID here means "signature id'. If you need to add your own rules, please refer to https://doc.emergingthreats.net/bin/view/Main/SidAllocation to choose your SID properly.

## Features

### Bypass

Firstly, "bypass" here means skipping the further steps[^2] during processing a packet, instead of skipping further rules, which is what I thought, misunderstoodly.

[^2]: "Suricata reads a packet, decodes it, checks it in the flow table...", refer to https://www.stamus-networks.com/blog/2016/09/28/suricata-bypass-feature

Bypassing is implemented in two ways in suricata, local and capture, local means the bypassing logic exists in suricata's source code. Capture means the logic is provided by the various "captures"(AF_PACKET, NFQ, etc...) in linux kernel, to utilize it, suricata just call a function provided by kernel and declare the rule it want to bypass.

## Developer resources

Developer wiki: https://redmine.openinfosecfoundation.org/projects/suricata/wiki
Roadmap: https://redmine.openinfosecfoundation.org/projects/suricata/roadmap

### Forefront topics

1. [Capture full session on alert](https://redmine.openinfosecfoundation.org/issues/120)

### Self building suricata

1. suricata honors the `autogen.sh`, `configure`, `make` build steps.
2. suricata relies on libhtp during building, we need to put the libhtp's source under suricata source root, then `./configure` will find it automatically
3. suricata relies on rust after 4.1, need to [install rust](https://www.rust-lang.org/tools/install) before `./configure`

### Source code

#### Thread vars, Thread modules and Thread module slots

In suricata's source code, `ThreadVars` represents a system thread, it also has a thread module slots which is a linked list of `TmModule`, on the other side, `ThreadVars` is wrapped by `Thread`, thereafter all of the `Thread`s are contained in `Threads`, meanwhile, `ThreadVars`s are connected to each other to a linked list too.

#### Program flow

```
RunModeDispatch() --> mode->RunModeFunc() --> Creates thread and assigns thread modules to slots, slot init == thread module init, slot func == thread func/pkt acqloop/management. Thread entry function is the tm_func() of ThreadVars, which usually the TmThreadsSlotPktAcqLoop()(src/tm-threads.c).
```

#### Packet arriving time

suricata relies on pcap to get the packet from network interface, when getting the packet, pcap will [populate the timestamp the packet arrives at the interface](https://github.com/the-tcpdump-group/libpcap/blob/fbcc461fbc2bd3b98de401cc04e6a4a10614e99f/pcap-netfilter-linux.c#L252), suricata gets this info and exposes it to us as well.

## Other resources

Suricata vs Snort: https://core.ac.uk/download/pdf/36699186.pdf
