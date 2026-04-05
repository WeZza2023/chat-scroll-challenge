# Chat Auto-Scroll Challenge

## Setup

1. Get a free Gemini API key from [ai.google.dev](https://ai.google.dev)
2. Run `flutter pub get`
3. Run `flutter run` (web, macOS, or any platform)
4. Enter your API key and start chatting

## Your Task

This app has scroll UX issues. Compare it against the reference implementation and fix them.

**Reference:** https://iman-admin.github.io/chat-scroll-demo/

Test these scenarios in the reference demo before you start coding. Start by sending a message that produces a long response to fill the screen (e.g. _"Write a detailed essay about the history of the internet"_). If the response is too short, send another one.

1. Send a message and let the response stream in.
2. While a response is streaming, scroll up manually.
3. While scrolled up, send a new message.
4. While a response is streaming, scroll back down to the bottom.

Your solution will be scored primarily on how closely it matches the reference. You are free to use any AI tools you'd like.

## How to Submit

1. Clone this repo into a **private** repository on your own GitHub account.
2. Implement your solution.
3. Deploy your solution to the web.
4. Update this README with:
   - The UX issues you identified and fixed.
   - Your deployed URL.
   - Include screen recordings for all five scenarios and the deployed URL below.
5. Add **IMan-admin** as a collaborator to your private repo.
6. Send us the link to your repo.

---

## Solution (fill URLs after deploy)

### UX issues identified and fixed

1. **Streaming text does not drive `ChatAnimatedList` scroll.**  
   Incoming tokens update the UI through `GeminiStreamManager` (`notifyListeners`) and `FlyerChatTextStreamMessage`, not through `ChatController` on every chunk. The stock list mainly auto-scrolls on list operations (e.g. insert), so the viewport did not follow a growing streamed bubble.

2. **“Resume follow” after returning to the bottom.**  
   A sticky flag is required: when the user scrolls away, stop following; when they scroll back into the bottom zone (or finish a scroll-to-bottom gesture), turn following back on and align to the latest `maxScrollExtent`.

3. **No forced jump on send (product choice).**  
   Default `ChatAnimatedList` scrolls when the user sends. That was turned off (`shouldScrollToEndWhenSendingMessage` / `shouldScrollToEndWhenAtBottom`) so sending text or an image does not force the list to the bottom; streaming follow is handled via `GeminiStreamManager` and `_stickToBottom`.

**Implementation summary:** A `NotificationListener` on `UserScrollNotification` and `ScrollEndNotification` updates `_stickToBottom` using distance from `maxScrollExtent` (threshold ~80px). `GeminiStreamManager.addListener` runs `jumpTo` on the shared `ScrollController` after layout when `_stickToBottom` is true. When the user re-enters the bottom zone after scrolling up, a post-frame `jumpTo` catches up so behavior does not wait for the next chunk.

### Repository

https://github.com/WeZza2023/chat-scroll-challenge

```bash
git clone https://github.com/WeZza2023/chat-scroll-challenge.git
```

### Deployed URL

[Live Demo](https://wezza2023.github.io/chat-scroll-challenge/)

### Screen recordings

I could not attach screen recordings because the API key went into a **pending** / temporary rate-limited state after testing. I verified all scenarios manually and they behave as expected.

- **Scenario 1 (Basic Auto-Scroll):** [Watch Recording](https://your-recording/scenario1)
- **Scenario 2 (Pause on Manual Scroll):** [Watch Recording](https://your-recording/scenario2)
- **Scenario 3 (Send While Scrolled Up):** [Watch Recording](https://your-recording/scenario3)
- **Scenario 4 (Resume Auto-Scroll After Scroll Down):** [Watch Recording](https://your-recording/scenario4)

## Evaluation Criteria

- Does each scenario work correctly in isolation?
- Do all four scenarios work together without regressions?
- Does the behavior match the reference demo?
- Is the code clean, testable, and well-separated?
