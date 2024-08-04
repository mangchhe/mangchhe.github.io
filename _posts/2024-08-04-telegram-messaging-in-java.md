---
layout: post
title: Telegram Messaging in Spring
categories: spring
tags: spring
---

Based on Telegram Bot API 7.8.

> [Telegram Bot API](https://core.telegram.org/bots/api)
>
> The Bot API is an HTTP-based interface

> [Authorizing your bot](https://core.telegram.org/bots/api#authorizing-your-bot)
>
> Each bot is given a unique authentication token when it is created.
 
> [BotFather](https://core.telegram.org/bots/features#botfather)
> 
> Telegram’s tool **for creating and managing bots.**
>
> Use the `/newbot` command to create a new bot. 
> 
> @BotFather will ask you for a name and username, then generate an authentication token for your new bot.
>
> - The name of your bot is displayed in contact details and elsewhere.
>
> - The username is a short name, used in search, mentions and t.me links.
>
> - The token is a string, like `110201543:AAHdqTcvCH1vGWJxfSeofSAs0K5PALDsaw`

> [Making requests](https://core.telegram.org/bots/api#making-requests)
>
> All queries to the Telegram Bot API must be served over HTTPS and need to be presented in this form: `https://api.telegram.org/bot<token>/METHOD_NAME`
>
> We support **GET** and **POST** HTTP methods. We support four ways of passing parameters in Bot API requests:

> [getUpdates](https://core.telegram.org/bots/api#getupdates)
>
> Use this method to receive incoming updates using long polling. Returns an Array of Update objects.
>
> `channel_post.sender_chat.id` is the chat ID.

> [sendMessage](https://core.telegram.org/bots/api#sendmessage)
>
> Use this method to send text messages.
>
> The `chat_id` and `text` fields are required.

**Implementation**

```java
@Slf4j
@Service
public class TelegramSchedulerClient {

    private final WebClient webClient;
    private final String chatId;

    public TelegramSchedulerClient(@Value("${telegram.bot.scheduler.token}") String botToken,
                                   @Value("${telegram.bot.scheduler.chat-id}") String chatId) {
        String url = "https://api.telegram.org/bot" + botToken + "/";
        this.webClient = WebClient.builder()
                .baseUrl(url)
                .defaultHeader("Content-Type", "application/json")
                .build();
        this.chatId = chatId;
    }

    public void success(String jobName, LocalDateTime startAt, LocalDateTime endAt) {
        long durationSec = Duration.between(startAt, endAt).getSeconds();
        try {
            webClient.post()
                    .uri("sendMessage")
                    .bodyValue(createMessagePayload(chatId, generateSuccessMessage(jobName, startAt, endAt, durationSec), "HTML"))
                    .retrieve()
                    .bodyToMono(String.class)
                    .block();
        } catch (WebClientResponseException | JsonProcessingException e) {
            log.info(LogMessage.format("Failed to send Telegram succeed message",
                    LogMessage.KeyValueData.of("jobName", jobName),
                    LogMessage.KeyValueData.of("startAt", startAt),
                    LogMessage.KeyValueData.of("endAt", endAt),
                    LogMessage.KeyValueData.of("durationSec", durationSec)
            ));
        }
    }

    public void fail(String jobName, Exception exception) {
        String errorPoint = null;
        String errorMessage = null;

        try {
            errorPoint = shortenText(exception.getStackTrace()[0].toString(), 500);
            errorMessage = shortenText(exception.getMessage(), 500);

            webClient.post()
                    .uri("sendMessage")
                    .bodyValue(createMessagePayload(chatId, generateFailMessage(jobName, errorPoint, errorMessage), "HTML"))
                    .retrieve()
                    .bodyToMono(String.class)
                    .block();
        } catch (WebClientResponseException | JsonProcessingException e) {
            log.info(LogMessage.format("Failed to send Telegram failed message",
                    LogMessage.KeyValueData.of("jobName", jobName),
                    LogMessage.KeyValueData.of("errorPoint", errorPoint),
                    LogMessage.KeyValueData.of("errorMessage", errorMessage)
            ));
        }
    }

    public static String generateSuccessMessage(String jobName, LocalDateTime startAt, LocalDateTime endAt, Long durationSec) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

        StringBuilder message = new StringBuilder();
        message.append("<b>작업이 성공적으로 완료되었습니다!</b>\n")
                .append("<b>작업 이름: </b>").append(jobName).append("\n")
                .append("<b>작업 시작 시간: </b>").append(startAt.format(formatter)).append("\n")
                .append("<b>작업 종료 시간: </b>").append(endAt.format(formatter)).append("\n")
                .append("<b>걸린 시간 (sec): </b>").append(durationSec).append("\n");

        return message.toString();
    }

    public static String generateFailMessage(String jobName, String errorPoint, String errorMessage) {
        StringBuilder message = new StringBuilder();
        message.append("<b>작업에 실패했습니다.</b>\n")
                .append("<b>작업 이름: </b>").append(jobName).append("\n")
                .append("<b>오류 지점: </b>\n<pre>").append(errorPoint).append("</pre>")
                .append("<b>오류 메시지: </b>\n<pre>").append(errorMessage).append("</pre>");

        return message.toString();
    }
}
```