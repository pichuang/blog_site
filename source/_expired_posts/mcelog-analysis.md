---
layout: post
title: "MCELOG Analysis"
description: "MCELOG Analysis"
date: 2018-03-18 00:00:00 +0800
updated: 2015-03-18 00:00:00 +0800
category: misc
tags: 
- misc
- mcelog
---

## Introduce
MCE - Machine Check Exception(Error) 是 Linux 用來檢查硬體錯誤的軟體，特別針對 CPU 和 Memory

## Case Study 
### Case 1 
```bash
Hardware event. This is not a software error.
MCE 0
mcelog: Family 6 Model 4d CPU: only decoding architectural errors
Hardware event. This is not a software error.
CPU 2 BANK 0 TSC 22366c36e58bc0
TIME 1520300677 Tue Mar  6 09:44:37 2018
MCG status:MCIP
MCi status:
Corrected error
Error enabled
MCA: External error
STATUS 9000000020000003 MCGSTATUS 4
MCGCAP 806 APICID 4 SOCKETID 0
CPUID Vendor Intel Family 6 Model 77
Unknown CPU type vendor 21 family 0 model 
```
- Root cause
  - Due to `mcelog: Family 6 Model 4d CPU: only decoding architectural errors`, mcelog cannot identify the CPU type.
- Solution
  - Upgrade mcelog to latest version
  - https://git.kernel.org/pub/scm/utils/cpu/mce/mcelog.git/

### Case 2
```bash
Hardware event. This is not a software error.
MCE 0
CPU 0 BANK 0
ADDR fef80000
TIME 1520309887 Tue Mar  6 04:18:07 2018
MCG status:
MCi status:
Uncorrected error
MCi_ADDR register valid
Processor context corrupt
MCA: Internal unclassified error: 410
Running trigger `unknown-error-trigger'
STATUS a600000007600410 MCGSTATUS 0
MCGCAP 806 APICID 0 SOCKETID 0
CPUID Vendor Intel Family 6 Model 77
```

### Case 3
```bash
mcelog: failed to prefill DIMM database from DMI data
Hardware event. This is not a software error.
MCE 0
CPU 0 BANK 0
ADDR fef80000
TIME 1521087195 Thu Mar 15 04:13:15 2018
MCG status:
MCi status:
Uncorrected error
MCi_ADDR register valid
Processor context corrupt
MCA: Internal unclassified error: 410
Running trigger `unknown-error-trigger'
STATUS a600000007600410 MCGSTATUS 0
MCGCAP 806 APICID 0 SOCKETID 0
CPUID Vendor Intel Family 6 Model 77
```
- Root cause
  - `mcelog: failed to prefill DIMM database from DMI data`
- Solution
  - This is a harmless warning message

## Useful command
- Check mcelog support the CPU typ or not
  - Non-support
```bash
root@localhost:# mcelog --is-cpu-supported
mcelog: Family 6 Model 4d CPU: only decoding architectural errors
```
  - Support 
```bash
root@localhost:# mcelog --is-cpu-supported
root@localhost:#
```
- Start mcelog daemon
`mcelog --daemon`
  - You need enable the service when booting

## Terminology
- MCE: Machine Check Exception(Error)
- MCA: Machine Check Architecture
- NMI: NMI notification of ECC errors
- MSRs: Machine Specific Register error cases

References: 
[Linux x86 machine check user space processing utility](1)
[Machine check exception - how to read and understand it?](2)

[1]: https://kernel.googlesource.com/pub/scm/utils/cpu/mce/mcelog/
[2]: https://superuser.com/questions/286504/machine-check-exception-how-to-read-and-understand-it
