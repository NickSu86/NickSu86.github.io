---
layout: post
title: AWS系列之Well Architecting Frameworks
---

终于决定要考AWS的Solution Architect Associate 了，仔细阅读了 [考纲](https://d1.awsstatic-china.com/training-and-certification/docs-sa-assoc/AWS_Certified_Solutions_Architect_Associate_Feb_2018_%20Exam_Guide_v1.5.2.pdf) 发现主要考点还是围绕着 [Well Architected](https://amazonaws-china.com/architecture/well-architected/) 的五个支柱理论，因此这次考试的准备就先从 [Well Architected Framework](https://d1.awsstatic-china.com/whitepapers/architecture/AWS_Well-Architected_Framework.pdf) 开始了。

首先简单介绍一下这个framework , 这是一套可供用户设计服务系统架构时参考或者评估用的标准。它提出了一系列的问题来帮助用户更全面的了解系统架构是否符合服务的最优使用方案， 同时也可以协助用户在现代云计算环境中设计和部署适合业务的系统架构。它的框架主要基于五个支柱，分别是：

1. Operational Excellence  The ability to run and monitor systems to deliver business value and to continually improve supporting processes and procedures
2. Security  The ability to protect information, systems, and assets while delivering business value through risk assessments and mitigation strategies.
3. Reliability  The ability of a system to recover from infrastructure or service disruptions, dynamically acquire computing resources to meet demand, and mitigate disruptions such as misconfigurations or transient network issues.
4. Performance Efficiency  The ability to use computing resources efficiently to meet system requirements, and to maintain that efficiency as demand changes and technologies evolve.
5. Cost Optimization   The ability to avoid or eliminate unneeded cost or suboptimal resources.

其实我相信各位在做架构设计的时候都会或多或少的考虑到上面几个方面 ， 但是真的这种东西没有总结出来一个成体系的东西，始终还是有点没底。

首先Operational Excellence，要求的是通过运行服务系统来传递商业价值，通过监控来不断改进流程和步骤的能力，近几年来很火的DevOps，其实就有这么一个味道，通过快速的迭代快速的试错，从而达到持续的改进。但是改进也是有个参考值的啊，这里就引入了监控了，通过监控来给不断的迭代和实验提供一个准确的反馈，然后根据反馈再不断的改进，这样才是一个架构生态。相信这个世界上没有一成不变的架构，随着业务的增长，技术的革新，架构不随着变化总会有一天跟不上需求从而被淘汰的。

第二点安全，通过风险评估和降级策略来保证信息，系统和资产的安全的能力。还记得CSDN的密码泄露事件么？还记得heart bleeding 的事件么？安全没做好，服务没办法有保障的提供，用户的信息都没有安全保障，又谈何发展？？？

第三点是可靠性，就是说在系统或者服务崩溃时可以自动恢复，在系统负载很高用户请求很大时能自动应用新的资源满足用户请求，在网络故障或者配置出错的时候能降低影响。这里可能涉及到 AutoScaling , CloudFormation之类的自动化工具，在发现系统问题时，可以快速自动的去相应调整。

第四点是高效的性能， 通过有效的利用计算资源来满足业务需求，在需求变更技术发展的时候可以有效的相应需求。

第五点是费用优化，通过减少和避免不必要的资源浪费来节省费用，老板省点钱，员工多点奖金，大家都happy 。

综合这几点来看呢，其实就是要把一个服务架构设计成一个有活力的系统，不是死的俄罗斯方块似的堆叠服务，而是说能自动相应部署或者减少资源，效率高，费用低，自发自动的一个架构。比如说刚才说的DevOps的持续集成，部署，反馈，优化等等。这里就要求用到 CloudWatch , AutoScaling , IAM , VPC 等等服务了。

