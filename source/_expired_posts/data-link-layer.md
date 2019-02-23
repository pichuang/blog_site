---
layout: post
title: "Data Link Layer"
date: 2015-01-05 10:07:00 +0800
updated: 2015-01-05 10:07:00 +0800
categories: Misc
tags:
- misc
---

transmission error detection
===

* checksum
* CRC

Reliable Data Transfer
===

* ARQ (Automatic Repeat reQuest) 

協議
===

* Go back N
* SAW (Start and Wait)
* Sliding Windows
* Selective Repeat
* Piggybacking
* Selective Acknowledgement

PPP (Point-to-Point Protocol)
===

* PPP 是基於 HDLC (High-Level DataLink Control) 設計, 主要用在 V.35 或 MODEM
* ADSL 的 PPPoE 及 VPN 的 PPTP 皆採用 PPP 協議封裝

CRC
===

* CRC-CCITT (x^16 + x^12 + x^5 + 1)
* 可檢測出所有隨機雙奇數位錯誤
* 可檢測出長度小於 16 位的錯誤達 100%
* 可檢測出長度大於 17 位的錯誤達 99.9969%
* 可檢測出長度大於 18 位的錯誤達 99.9985%
