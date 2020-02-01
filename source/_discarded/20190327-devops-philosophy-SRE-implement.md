layout: draft
title: DevOps 是哲學 SRE 是實踐
author: Phil Huang
date: 2019-03-27 10:10:42
tags:
---
## What is SRE?

如果不想看英文的話，可以參考 [好文翻譯 你在找的是 SRE 還是 DevOps？][3]

### Agenda of SRE: Measuring / Managing Reliability
- Targeting Reliability (定義可靠性)
- Operating for Reliability (運行可靠性)
- Choosing a Good SLI (選擇好的 SLI)
- Developing SLOs and SLIs (開發 SLO 和 SLI)
- Quantifying Risks to SLOs (量化 SLO 風險)
- Consequences of SLO Misses (SLO 未命中的後果)

> SLO: Service Level Objectives 具體目標  
SLI: Service Level Indicator 量化指標  
SLA: Service Level Agreement 協議

### What's the Difference between DevOps and SRE?

5 Key for DevOps philosophy
- Reduce Organization Silos
- Accept Failure as Normal
- Implement Gradual Change
- Leverage Tooling & Automation
- Measure Everything

### class SRE implements DevOps

```java
class SRE implemnts DevOps
```

- Reduce Organization Silos => Share ownership
- Accept Failure as Normal => SLOs & Blameless PMs
- Implement Gradual Change => Reduce costs of failure
- Leverage Tooling & Automation => Automate this year's job away
- Measure Everything => Measure toil and reliability

## References
- [What's the Difference Between DevOps and SRE? (class SRE implements DevOps)][1]
- [Now SRE Everyone Else with CRE! (class SRE implements DevOps)][2]
- [好文翻譯 你在找的是 SRE 還是 DevOps？][3]
- [深度剖析什麼是 SLI、SLO和SLA？][4]

[1]: https://www.youtube.com/watch?v=uTEL8Ff1Zvk
[2]: https://www.youtube.com/watch?v=GQPzaq-owYM
[3]: https://medium.com/kkstream/%E5%A5%BD%E6%96%87%E7%BF%BB%E8%AD%AF-%E4%BD%A0%E5%9C%A8%E6%89%BE%E7%9A%84%E6%98%AF-sre-%E9%82%84%E6%98%AF-devops-2ded43c2852
[4]: https://www.itread01.com/content/1545871177.html
[5]: https://medium.com/kkstream/%E5%A5%BD%E6%96%87%E7%BF%BB%E8%AD%AF-%E4%BD%A0%E5%9C%A8%E6%89%BE%E7%9A%84%E6%98%AF-sre-%E9%82%84%E6%98%AF-devops-2ded43c2852


