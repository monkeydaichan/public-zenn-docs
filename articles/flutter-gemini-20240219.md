---
title: "FlutterでGeminiとチャットしてみた"
emoji: ":🤖:"
type: "tech"
topics: ["Flutter","Dart","Gemini", "Google", "AI"]
published: true
---

Flutter 3.19のリリースノートを見ていると DartからGeminiが使えるようになっていたのでGeminiとチャットするアプリを作りました!
https://medium.com/flutter/whats-new-in-flutter-3-19-58b1aae242d2

めちゃくちゃ簡単にできた．．．
API無料だったんだけど．．．

コードここに置いてます
https://github.com/monkeydaichan/flutter_gemini


## GeminiのAPI Keyを取得
https://aistudio.google.com/app/prompts/new_chat


## パッケージ追加
- google_generative_ai https://pub.dev/packages/google_generative_ai
- flutter_chat_ui https://pub.dev/packages/flutter_chat_ui
- flutter_dotenv https://pub.dev/packages/flutter_dotenv
- flutter_riverpod https://pub.dev/packages/flutter_riverpod

## コード書いてく
```sh
$ flutter doctor                                                                                                                                           
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.19.0, on macOS 13.5.2 22G91 darwin-arm64, locale ja-US)
[✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
[✓] Xcode - develop for iOS and macOS (Xcode 15.0.1)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2022.3)
[✓] VS Code (version 1.86.2)
[✓] Connected device (3 available)
[✓] Network resources
```
```dart
import 'package:flutter/material.dart';
import 'package:flutter_chat_types/flutter_chat_types.dart' as types;
import 'package:flutter_chat_ui/flutter_chat_ui.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:google_generative_ai/google_generative_ai.dart';

final googleAIStudioAPIKey = dotenv.get('GOOGLE_AI_STUDIO_KEY');

final generativeModelProvider =
    Provider<GenerativeModel>((ref) => GenerativeModel(apiKey: googleAIStudioAPIKey, model: 'gemini-pro'));

final messagesNotifier = StateNotifierProvider<MessagesNotifier, List<types.Message>>((ref) {
  return MessagesNotifier(ref.watch(generativeModelProvider).startChat());
});

class MessagesNotifier extends StateNotifier<List<types.Message>> {
  MessagesNotifier(this.chat) : super([]);

  late final ChatSession chat;

  void addMessage(types.User author, String text) {
    final timeStamp = DateTime.now().millisecondsSinceEpoch.toString();
    final message = types.TextMessage(author: author, id: timeStamp, text: text);
    state = [message, ...state];
  }

  Future<void> askGemini(String question) async {
    addMessage(me, question);
    final content = Content.text(question);
    try {
      final response = await chat.sendMessage(content);
      final message = response.text ?? 'Retry later...';
      addMessage(gemini, message);
    } on Exception {
      addMessage(gemini, 'Retry later...');
    }
  }
}

const gemini = types.User(id: 'gemini');
const me = types.User(id: 'user');

class ChatRoomScreen extends ConsumerWidget {
  const ChatRoomScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messages = ref.watch(messagesNotifier);
    return Scaffold(
      body: Chat(
        user: me,
        messages: messages,
        onSendPressed: (a) {
          ref.read(messagesNotifier.notifier).askGemini(a.text);
        },
      ),
    );
  }
}
```

## 大谷翔平はなにが凄い?

![](/images/flutter-gemini-20240219/1.png)


