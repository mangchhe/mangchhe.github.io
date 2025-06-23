---
layout: post
title: How to Increase Datadog Log Index Retention Period
categories: devops
tags: devops
---

Datadog에서 **Log Index의 보관 기간(retention period)**을 늘리려면 다음 설정을 하면 된다.  
기본 보관 기간은 **15일**이며, 별도 설정 없이는 이를 초과할 수 없다.

#### 로그 인덱스 설정

**Logs > Configuration > Indexes**로 들어가서, 보관 기간을 변경하고 싶은 인덱스를 선택한다.

**Set Index Retention Period** 항목이 있는데, 기본적으로는 비활성화되어 있을 수 있다.

#### 보관 기간 설정 기능 활성화

이 항목을 사용하려면 설정에서 따로 기능을 켜야 한다.

- **General > Preferences** 메뉴로 이동
- 아래 항목을 켠다:  
  > **Out-of-Contract Retention Periods for Log Indexes**

이걸 켜면 인덱스별로 보관 기간을 **15일 초과로 설정**할 수 있다.

![datadog-log-retention](/assets/postImages/HowToIncreaseDatadogLogIndexRententionPeriod/datadog-log-retention.png)