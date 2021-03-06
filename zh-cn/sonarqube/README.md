# SonarQube

>基于docker-compose安装

```shell
version: "3"

services:
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
    networks:
      - sonarqube_network

  sonarqube:
    image: sonarqube:lts
    environment:
      sonar.jdbc.url: jdbc:postgresql://db:5432/sonar
      sonar.jdbc.username: sonar
      sonar.jdbc.password: sonar
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
    networks:
      - sonarqube_network

networks:
  sonarqube_network:

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

---

## 如何使用`SonarQube`开发高质量的软件

使用诸如SonarQube 之类的工具，可以为程序员带来更高的质量和更高的生产力，并促进代码的某种“标准化”，这是一个非常重要的因素，因为软件产品的某些部分通常在不同的人之间共享。

通过对源代码的静态分析，SonarQube 衡量可靠性、可维护性、安全性、复杂性、测试覆盖率和代码重复的级别，并根据发现的缺陷的严重程度分配一个分数。

当软件质量保持较高时，可以保证更大的安全性，并可以在产品的整个生命周期中降低成本，减少维护时间，为生产者、客户和金融家创造价值。

1. 如何量化软件的质量

软件的高质量与以下因素有关：

- 减少测试和交付时间
- 减少超过 50% 的维修和修改
- 提高客户满意度
- 发布后降低维护成本
- 减少项目合同纠纷
- 减少取消的项目
- 提高可靠性
- 减少已发布应用程序中的安全漏洞

2. SonarQube 规则的类别

SonarQube 使用分为四种类型的规则来分析代码：代码异味、错误、漏洞和安全热点。

- 代码味道类别包括显示编程缺陷，但没有揭示错误，因此不会影响软件的实际正确性，但使代码少维护的特点。
- 错误是程序意外或不正确行为所依赖的错误，降低了程序的可靠性。
- 漏洞是可以被利用来破坏系统安全级别的弱点。
- 安全热点是没有问题但可能成为漏洞的弱点。示例包括错误的 cookie 配置、非标准加密算法的使用以及无法建立安全连接的协议的使用。

还可以通过 SonarQube 平台创建新规则，并且可以选择一组规则与每种语言相关联。


3. 严重程度

在分析过程中，每次代码片段违反规则时都会定义问题。每个问题都与不同的严重程度相关：

- 拦截器：一个可以改变程序行为的错误。必须检查代码。
- 严重：不太可能改变程序行为或安全缺陷的错误。代码来自于审查。
- 主要：降低开发人员生产力的代码缺陷，例如重复的代码、未使用的参数和变量
- 次要：代码缺陷会略微降低开发人员的生产力，例如太长的行和少于三个案例的开关。
- 信息：这不是真正的软件缺陷，例如争用“TODO”的评论或使用已弃用的结构。

4. 五状态

一个问题，一旦创建，可以呈现五种状态：在它创建后立即(1) open，当用户表示它是一个有效问题时它变为(2) 确认，(3)当用户表示在不应再次考虑下一次分析，因为已经进行了更改，(4)当问题报告为已解决但尚未真正纠正时重新打开，以及(5)当 SonarQube 不再将其识别为问题时关闭。问题在修复后或因为识别它的相关规则不再可用时被关闭，已删除。

5. 指标的评级

用于定义代码质量的指标是复杂性、重复性、可维护性、可靠性、安全性、大小和测试覆盖率。

可靠性等级

- A = 0 错误
- B = 至少一个小错误
- C = 至少一个主要错误
- D = 至少一个严重错误
- E = 至少一个错误拦截器

安全等级

- A = 0 漏洞
- B = 至少有轻微的漏洞
- C = 至少一个主要漏洞
- D = 至少一个严重漏洞
- E = 至少一个拦截器漏洞

技术债务评级

这取决于“技术债务比率”（TD 比率），即债务与预计开发时间的比率，计算为 LOCx30min。

- A = TD 比率 <= 5%
- B = TD 比率在 6% 到 10% 之间
- C = TD 比率介于 10% 和 20% 之间
- D = TD 比率在 21% 和 50% 之间
- E = TD 比率> = 50%


高质量软件的生产成本更低
如果您相信持续集成，则不能忽视 SonarQube 的使用，因为它允许您在每次更改后检查代码，从而始终保持软件的高品质。