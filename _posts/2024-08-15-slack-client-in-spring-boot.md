---
layout: post
title: Slack Client in Spring Boot
categories: spring
tags: spring
---

**Slack App**

앱 추가 → Incoming WebHooks 구성 → Slack에 추가 → 메시지 전송할 채널 선택 후 앱 추가 → 앱 이름/아이콘 설정 → 웹훅 URL 복사

```
curl -X POST --data-urlencode \
"payload={
    \"channel\": \"#my-channel-here\", 
    \"username\": \"webhookbot\", 
    \"text\": \"이 항목은 #my-channel-here에 포스트되며 webhookbot이라는 봇에서 제공됩니다.\", 
    \"icon_emoji\": \":ghost:\"
}" \
https://hooks.slack.com/services/{Team ID}/{Integration ID}/{Secret Key}
```

**Dependency**

```gradle
// https://mvnrepository.com/artifact/com.slack.api/slack-api-client/1.40.3
implementation("com.slack.api:slack-api-client:1.40.3")
```

**Implementation**

```kotlin
@Slf4j
@Service
public class SlackClient {
    @Value("${webhook.slack.app.webhook-url}")
    private String SLACK_APP_WEBHOOK_URL;

    private final Slack slackClient = Slack.getInstance();

    public void send(Exception e) {
        try {
            slackClient.send(SLACK_APP_WEBHOOK_URL, payload(p -> p
                .attachments(List.of(
                    Attachment.builder().color(convertToHex(Color.RED))
                        .fields(
                            List.of(
                                generateSlackField("Summary", e.getMessage()),
                                generateSlackField("Details", formatStackTrace(e.getStackTrace()))
                            )
                        ).build())))
            );
        } catch (IOException ex) {
            log.warn("App Message Slack failed.");
        }
    }

    private Field generateSlackField(String key, String value) {
        return Field.builder()
                .title(key)
                .value(value)
                .valueShortEnough(false)
                .build();
    }

    private String convertToHex(Color color) {
        return String.format("#%02x%02x%02x", color.getRed(), color.getGreen(), color.getBlue());
    }

    private String formatStackTrace(StackTraceElement[] stackTrace) {
        return Arrays.stream(stackTrace)
                .map(element -> "> " + element.toString())
                .collect(Collectors.joining("\n"));
    }
}
```

**Results**

![slack-error-message](/assets/postImages/SlackClientInSpringBoot/slack-error-message.png)