Canon EDSDK（EOS Digital Software Development Kit）でカメラをリモート制御したいとき、いくつかハマりどころがあるので備忘録。

## (1) TAKE_PICTURE_CARD_NGで撮影できない

`kEdsPropID_SaveTo`を`kEdsSaveTo_Host`に設定すると、撮影された画像はホストPC側に転送されるにも関わらず、CARD_NGというエラーが出る。

このケースでは、SetCapacityを撮影前に行う必要がある。PC側に十分な容量があると教えてあげるためだと思われる。

```cpp
EdsCapacity capacity = { 0x7FFFFFFF, 0x1000, 1 };
err = EdsSetCapacity(camera, capacity);
```

## (2) ObjectEventだけが呼ばれない

ホストPC側に画像を転送する際、ObjectEventHandlerで非同期イベントを受信して画像を受け取るのだが、Windowsではこのイベントが全くこないことがある。（しかも、ParameterEventやStateEventはきちんと呼ばれるにも関わらず…。）

この場合は、リモートレリーズ制御直後に以下のコードを挿入し、Win32APIでメッセージ受信をする必要がある。（イベント自体は非同期でハンドラが呼ばれる。）

```cpp
#ifdef WIN32
		MSG msg;
		while (GetMessage(&msg, NULL, NULL, NULL)) {
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
#endif
```

このwhile文中では、実際に欲しいイベント（例えばkEdsObjectEvent_DirItemRequestTransfer）以外にもいろいろ受信するので、実際に目的のイベントが受信されたら適宜breakする必要がある。

## 参考

- https://stackoverflow.com/questions/22965982/canon-edsdk-saving-image-in-my-pc/22976188
- https://stackoverflow.com/questions/6891693/taking-picture-with-edsdk-and-retrieving-it-immediately/10428426
- http://songuke.blogspot.com/2012/05/short-note-about-using-canon-edsdk.html

（この辺の情報、公式ドキュメントにも載せてくれたらホントにいいのになぁ…。）
