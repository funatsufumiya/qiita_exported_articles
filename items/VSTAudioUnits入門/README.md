<div>
<img alt="VST" src="http://imgur.com/Im7Iggn.jpg" width="160px"/>

<img alt="AU" src="http://imgur.com/i0eEgHY.jpg" height="130px"/>
</div>

VSTとAudioUnitは日本語ドキュメントが少ないので、まとめておきます。

# VST

<img alt="VST" src="http://imgur.com/Im7Iggn.jpg" width="160px"/>

## VSTとは

VSTは、

_Steinberg's **V**irtual **S**tudio **T**echnology_

の略です。

名前の通り、Steinberg（スタインバーグ）社が提供している仮想音響技術のことです。

仕組みは、C++のダイナミックライブラリを使い、 **ホストアプリケーション**と呼ばれるアプリケーションに動的に組み込まれ、リアルタイム／非リアルタイムにフィルタや仮想楽器をレンダリングすることができます。

クロスプラットフォームな技術ですが、C++をプラットフォームごとにコンパイルする必要があり、特にGUIには注意する必要があります。

なお、楽器（シンセサイザー）として機能するVSTは、特に **VSTi**と呼ばれます。

## SDKのダウンロード

- Developer Site (Steinberg)
http://www.steinberg.net/en/company/developers.html

### 開発者アカウント登録

以下のURLで開発者として登録をする。

http://www.steinberg.net/nc/en/company/developers/sdk_download_portal/create_3rd_party_developer_account.html

> 注：登録する時にユーザ名を登録するが、どうもメールアドレスをログインユーザ名として使う？

### 既にアカウントを持っている場合

以下のURLでアカウントを拡張する。

http://www.steinberg.net/nc/en/company/developers/sdk_download_portal/extend_existing_mysteinberg_account.html

### ダウンロードページに戻る

以下のサイトに再度アクセスすると、ダウンロード出来るようになる。後は指示に従えばOK。（ちなみにASIO SDKもダウンロードできる。）

http://www.steinberg.net/en/company/developers.html


## SDKを展開してみる

ダウンロードしたzipファイルを展開すると、「VST3 SDK」というフォルダができる。中身は以下のようになっている。

<div style="background:#EEEEEE; padding:10px;">

 <i><b>base</b></i><br>
<span style="color:gray;">――</span> mac<br>
<span style="color:gray;">――</span> source<br>
<span style="color:gray;">――</span> win<br>
 <i><b>bin</b></i><br>
<span style="color:gray;">――</span> Mac OS X<br>
<span style="color:gray;">――</span> Windows 32 bit<br>
<span style="color:gray;">――</span> Windows 64 bit<br>
 <i><b>doc</b></i><br>
<span style="color:gray;">――</span> artwork<br>
<span style="color:gray;">――</span> base<br>
<span style="color:gray;">――</span> basemodule<br>
<span style="color:gray;">――</span> doxygfx<br>
<span style="color:gray;">――</span> gfx<br>
<span style="color:gray;">――</span> vstexamples<br>
<span style="color:gray;">――</span> vstinterfaces<br>
<span style="color:gray;">――</span> vstsdk<br>
 <i><b>pluginterfaces</b></i><br>
<span style="color:gray;">――</span> base<br>
<span style="color:gray;">――</span> gui<br>
<span style="color:gray;">――</span> test<br>
<span style="color:gray;">――</span> vst<br>
<span style="color:gray;">――</span> vst2.x<br>
 <i><b>public.sdk</b></i><br>
<span style="color:gray;">――</span> <b>samples</b><br>
<span style="color:gray;">――――</span> <b>vst</b><br>
<span style="color:gray;">――――――</span> InterAppAudio<br>
<span style="color:gray;">――――――</span> adelay<br>
<span style="color:gray;">――――――</span> again<br>
<span style="color:gray;">――――――</span> common<br>
<span style="color:gray;">――――――</span> hostchecker<br>
<span style="color:gray;">――――――</span> mac<br>
<span style="color:gray;">――――――</span> mda<span style="color:gray;">―</span>vst3<br>
<span style="color:gray;">――――――</span> note_expression_synth<br>
<span style="color:gray;">――――――</span> note_expression_text<br>
<span style="color:gray;">――――――</span> pitchnames<br>
<span style="color:gray;">――――――</span> validator<br>
 <i><b>vstgui.sf</b></i><br>
<span style="color:gray;">――</span> drawtest<br>
<span style="color:gray;">――</span> libpng<br>
<span style="color:gray;">――</span> vstgui<br>
<span style="color:gray;">――</span> zlib<br>
 <i><b>vstgui4</b></i><br>
<span style="color:gray;">――</span> <b>vstgui</b><br>
<span style="color:gray;">――――</span> Documentation<br>
<span style="color:gray;">――――</span> doxygen<br>
<span style="color:gray;">――――</span> <b>ide</b><br>
<span style="color:gray;">――――――</span> codeblocks<br>
<span style="color:gray;">――――――</span> visualstudio<br>
<span style="color:gray;">――――――</span> xcode4<br>
<span style="color:gray;">――――</span> <b>lib</b><br>
<span style="color:gray;">――――――</span> animation<br>
<span style="color:gray;">――――――</span> controls<br>
<span style="color:gray;">――――――</span> <b>platform</b><br>
<span style="color:gray;">――――――――</span> <b>mac</b><br>
<span style="color:gray;">――――――――――</span> carbon<br>
<span style="color:gray;">――――――――――</span> cocoa<br>
<span style="color:gray;">――――――――――</span> ios<br>
<span style="color:gray;">――――――――</span> <b>win32</b><br>
<span style="color:gray;">――――――――――</span> direct2d<br>
<span style="color:gray;">――――</span> plugin<span style="color:gray;">―</span>bindings<br>
<span style="color:gray;">――――</span> tests<br>
<span style="color:gray;">――――</span> tutorial<br>
<span style="color:gray;">――――</span> uidescription<br>

</div>

### 内容

内容を見てみると、サンプルコードやGUI、IDEに関するリソースも提供されています。また、doxygenなどによるドキュメントも入っているようです。

### サンプルコード

上記のサンプルコードから、「pitchnames」というプロジェクトのコードを部分的に紹介します。

#### pithnames.h

```cpp
// Project     : VST SDK
// Version     : 3.6.0
//
// Category    : Examples
// Filename    : public.sdk/samples/vst/pitchnames/source/pitchnames.h
// Created by  : Steinberg, 12/2010

/* ============ */
/* === 中略 === */
/* ============ */

#ifndef __pitchnames__
#define __pitchnames__

#include "public.sdk/source/vst/vsteditcontroller.h"
#include "public.sdk/source/vst/vstaudioeffect.h"
#include "base/source/fstring.h"
#include "base/source/tdictionary.h"
#include "vstgui/plugin-bindings/vst3editor.h"

namespace Steinberg {
namespace Vst {

//-----------------------------------------------------------------------------
class PitchNamesController : public EditControllerEx1, public VSTGUI::VST3EditorDelegate
{
public:
	tresult PLUGIN_API initialize (FUnknown* context);

	virtual tresult PLUGIN_API getUnitByBus (MediaType type, BusDirection dir, int32 busIndex, int32 channel, UnitID& unitId /*out*/);

	virtual IPlugView* PLUGIN_API createView (FIDString name);

	VSTGUI::CView* createCustomView (VSTGUI::UTF8StringPtr name, const VSTGUI::UIAttributes& attributes, VSTGUI::IUIDescription* description, VSTGUI::VST3Editor* editor);

	static FUnknown* createInstance(void*) { return (IEditController*)new PitchNamesController (); }
	static FUID cid;
protected:
	TDictionary<int16, String> pitchNames;
};

//-----------------------------------------------------------------------------
class PitchNamesProcessor : public AudioEffect
{
public:
	PitchNamesProcessor ();

	tresult PLUGIN_API initialize (FUnknown* context);
	tresult PLUGIN_API process (ProcessData& data);

	static FUnknown* createInstance(void*) { return (IAudioProcessor*)new PitchNamesProcessor (); }
	static FUID cid;
};

}} // namespaces

#endif

```

まず、 #include を見てみましょう。

```cpp
#include "public.sdk/source/vst/vsteditcontroller.h"
#include "public.sdk/source/vst/vstaudioeffect.h"
#include "base/source/fstring.h"
#include "base/source/tdictionary.h"
#include "vstgui/plugin-bindings/vst3editor.h"
```

単純化すると（イメージのコードです。動きません。）、

```cpp
include "vsteditcontroller";
include "vstaudioeffect";
include "base -> source -> fstring";
include "base -> source -> tdictionary";
include "vstgui -> vst3editor";
```

見る感じ、 **VSTEditController** というものが大事のようですね。
このPitchNamesというのはエフェクトのようで、 **VSTAudioEffect** をインクルードするとエフェクトが作成できるようですね。

**PitchNamesController**クラスがGUIで、 **PitchNamesProcessor**が音声を生成しています。特に **process**メソッドが重要です。ここに生成処理を記述します。

では次に本体を見てみましょう。

#### pitchnames.cpp


```cpp
// Project     : VST SDK
// Version     : 3.6.0
//
// Category    : Examples
// Filename    : public.sdk/samples/vst/pitchnames/source/pitchnames.h
// Created by  : Steinberg, 12/2010

/* ============ */
/* === 中略 === */
/* ============ */

#include "pitchnames.h"
#include "pitchnamesdatabrowsersource.h"
#include "pluginterfaces/base/ustring.h"

using namespace VSTGUI;

/* ============ */
/* === 中略 === */
/* ============ */

namespace Steinberg {
namespace Vst {

/* ============ */
/* === 中略 === */
/* ============ */

tresult PLUGIN_API PitchNamesController::initialize (FUnknown* context)
{
	tresult result = EditControllerEx1::initialize (context);
	if (result == kResultOk)
	{
	#if MULTI_CHANNEL_SCENARIO
		addUnit (new Unit (String ("Root"), kRootUnitId, kNoParentUnitId));
		addUnit (new Unit (String ("Channel 0"), kChannel0UnitId, kRootUnitId, kProgramList0ID));
		addUnit (new Unit (String ("Channel 1"), kChannel1UnitId, kRootUnitId, kProgramList1ID));

		ProgramListWithPitchNames* list = new ProgramListWithPitchNames (String ("ProgramList 0"), kProgramList0ID, kChannel0UnitId);
		list->addProgram (String ("Init 1"));
		list->setPitchName (0, 0, String ("Channel 0 First Item"));
		addProgramList (list);
		parameters.addParameter (list->getParameter ());

		list = new ProgramListWithPitchNames (String ("ProgramList 1"), kProgramList1ID, kChannel1UnitId);
		list->addProgram (String ("Init 1"));
		list->setPitchName (0, 0, String ("Channel 1 First Item"));
		addProgramList (list);
		parameters.addParameter (list->getParameter ());

/* ============ */
/* === 中略 === */
/* ============ */

}


/* ============ */
/* === 中略 === */
/* ============ */

//-----------------------------------------------------------------------------
IPlugView* PLUGIN_API PitchNamesController::createView (FIDString name)
{
	if (ConstString (name) == ViewType::kEditor)
	{
		return new VST3Editor (this, "PitchNamesEditor", "pitchnames.uidesc");
	}
	return 0;
}

//-----------------------------------------------------------------------------
CView* PitchNamesController::createCustomView (UTF8StringPtr name, const UIAttributes& attributes, IUIDescription* description, VST3Editor* editor)
{
#if MULTI_CHANNEL_SCENARIO
	if (ConstString (name) == "PitchNamesDataBrowser")
	{
		IDataBrowser* dataSource = new PitchNamesDataBrowserSource (this, kChannel0UnitId);
		return new CDataBrowser (CRect (0, 0, 100, 100), 0, dataSource, CDataBrowser::kDrawRowLines|CDataBrowser::kDrawColumnLines|CDataBrowser::kDrawHeader|CDataBrowser::kVerticalScrollbar);
	}
#else

/* ============ */
/* === 中略 === */
/* ============ */

}


/* ============ */
/* === 中略 === */
/* ============ */

//-----------------------------------------------------------------------------
tresult PLUGIN_API PitchNamesProcessor::initialize (FUnknown* context)
{
	tresult result = AudioEffect::initialize (context);
	if (result == kResultOk)
	{
		addAudioOutput (USTRING("Audio Output"), SpeakerArr::kStereo);
	#if MULTI_CHANNEL_SCENARIO
		addEventInput (USTRING("Event Input"), 2);
	#else
		addEventInput (USTRING("Event Input"), 1);
	#endif
	}
	return result;
}

//-----------------------------------------------------------------------------
tresult PLUGIN_API PitchNamesProcessor::process (ProcessData& data)
{
	if (data.numSamples && data.numOutputs)
	{
		for (int32 i = 0; i < data.outputs[0].numChannels; i++)
			memset (data.outputs[0].channelBuffers32[i], 0, data.numSamples * sizeof (float));
		data.outputs[0].silenceFlags = 0x3;
	}
	return kResultOk;
}

}} // namespaces

```

特に大切な部分は、

```cpp
tresult PLUGIN_API PitchNamesProcessor::process (ProcessData& data)
{
	if (data.numSamples && data.numOutputs)
	{
		for (int32 i = 0; i < data.outputs[0].numChannels; i++)
			memset (data.outputs[0].channelBuffers32[i], 0, data.numSamples * sizeof (float));
		data.outputs[0].silenceFlags = 0x3;
	}
	return kResultOk;
}
```

ですね。ここで音声生成のアルゴリズムを記述しています。

**ProcessData& data** が、出力されるデータ列を記述するためのアドレスで、 **data.outputs[0]** が左信号、 **data.outputs[1]** が右信号です。サラウンドになるともっと増えます。

実際に出力しているのは、以下の１行です。

```cpp
memset (data.outputs[0].channelBuffers32[i], 0, data.numSamples * sizeof (float));
```

memsetは、 **memset(*buf, ch, n)** とすると、「bufからnバイト分だけchをセットする」という関数です。

つまり、

```cpp
*buf = data.outputs[0].channelBuffers32[i];
ch = 0;
n = data.numSamples * sizeof (float);
```

とは、 **「出力データの左音声」** の`channelBuffers32`から、 **「出力データのサンプル数 × 浮動点小数のサイズ」 ** 分だけ **「0」** をセットする** という意味です。

つまり、`channelBuffers32`からサンプル数だけ出力データを全て0にするということです。

もしこれが正弦波（sin関数）ならば、サンプル数だけ`sin(n/x)`の値をセットすることになります。

少し分かって貰えたでしょうか。AudioUnitもほぼ同じ要領です。


# AU (Audio Units)

<img alt="AU" src="http://imgur.com/i0eEgHY.jpg" />

## AUとは

AU (AudioUnits) とは、Mac OS Xにおける音声処理を仮想化する技術です。

基本的にはVST/VSTiと変わりませんが、仕組みは異なります。

VSTがダイナミックライブラリ（dll/dylib）なのに対して、AUはComponentという形式を使います。

VSTがクロスプラットフォームなのに対し、AUはMacでしか使えません。その代わり、OSに近い低レベルレイヤーにあるので、非常にクオリティが高いです。

## 前提

まず大前提として、Macが必要です。おそらくMax OS X10.5以降なら大丈夫だと思います。

ちなみに私はMac OS X 10.9.3/MacbookAir Late 2010を使っています。

AUに対応しているかを確認するには、ターミナルを開いて、

```sh
ls /Library/Audio/Plug-Ins/

# Components/  HAL/  MAS/  VST/  VST3/
```

というように、 **Components**フォルダが存在するかを確認して下さい。ちなみにこの **/Library/Audio/Plug-Ins/Components/** にAUプラグインを配置すると認識されます。

蛇足ですが、 **/Library/Audio/Plug-Ins/VST/** にVSTプラグインを配置すると、VSTとして認識されます。

## SDK・サンプルコードの準備

AUのSDKはMacにプリインストールされています。Macのバージョンによって微妙に違うので、その都度確認して下さい。

AUのサンプルコードは、Apple Developer Libraryからダウンロードしてください。
https://developer.apple.com/library/mac/samplecode/sc2195/Introduction/Intro.html

分かりづらいですが、赤い円のところをクリックするとダウンロードできます。

![instruction](http://imgur.com/6XOp6fw.jpg)

## サンプルコードを見てみる

さっきと順序が逆ですが、ライブラリを理解するためにサンプルコードを見てみましょう。

#### SinSynth.h

```cpp
/*
     File: SinSynth.h
 Abstract: SinSynth.h
  Version: 1.0.1

======== 中略 ==========

 Copyright (C) 2014 Apple Inc. All Rights Reserved.
 
*/

#include "AUInstrumentBase.h"
#include "SinSynthVersion.h"

static const UInt32 kNumNotes = 12;

struct TestNote : public SynthNote
{
	virtual					~TestNote() {}

	virtual bool			Attack(const MusicDeviceNoteParams &inParams)
								{ 

/* ======== 中略 ========== */

									double sampleRate = SampleRate();
									phase = 0.;
									amp = 0.;
									maxamp = 0.4 * pow(inParams.mVelocity/127., 3.); 
									up_slope = maxamp / (0.1 * sampleRate);
									dn_slope = -maxamp / (0.9 * sampleRate);
									fast_dn_slope = -maxamp / (0.005 * sampleRate);
									return true;
								}
	virtual void			Kill(UInt32 inFrame); // voice is being stolen.
	virtual void			Release(UInt32 inFrame);
	virtual void			FastRelease(UInt32 inFrame);
	virtual Float32			Amplitude() { return amp; } // used for finding quietest note for voice stealing.
	virtual OSStatus		Render(UInt64 inAbsoluteSampleFrame, UInt32 inNumFrames, AudioBufferList** inBufferList, UInt32 inOutBusCount);

	double phase, amp, maxamp;
	double up_slope, dn_slope, fast_dn_slope;
};

class SinSynth : public AUMonotimbralInstrumentBase
{
public:
								SinSynth(AudioUnit inComponentInstance);
	virtual						~SinSynth();
				
	virtual OSStatus			Initialize();
	virtual void				Cleanup();
	virtual OSStatus			Version() { return kSinSynthVersion; }

	virtual AUElement*			CreateElement(			AudioUnitScope					scope,
											  AudioUnitElement				element);

	virtual OSStatus			GetParameterInfo(		AudioUnitScope					inScope,
														AudioUnitParameterID			inParameterID,
														AudioUnitParameterInfo &		outParameterInfo);

	MidiControls*				GetControls( MusicDeviceGroupID inChannel)
	{
		SynthGroupElement *group = GetElForGroupID(inChannel);
		return (MidiControls *) group->GetMIDIControlHandler();
	}
	
private:
	
	TestNote mTestNotes[kNumNotes];
};

```

よく見てみると、 **AUInstrumentBase**というクラスをベースにしているのがわかります。SinSynthはシンセサイザーだということがわかりますね。

ここでちょっと、AUInstrumentBaseがどういうものなのか見てみましょう。

AUInstrumentBaseは、 */CoreAudio/AudioUnits/AUPublic/AUInstrumentBase/AUInstrumentBase.h*に存在します。

```cpp
/*
     File: AUInstrumentBase.h 
 Abstract:  Part of CoreAudio Utility Classes  
  Version: 1.0.4 
  
=========== 中略 ===========
  
 Copyright (C) 2013 Apple Inc. All Rights Reserved. 
  
*/
#ifndef __AUInstrumentBase__
#define __AUInstrumentBase__
 
#include <vector>
#include <stdexcept>
#include <AudioUnit/AudioUnit.h>
#include <CoreAudio/CoreAudio.h>
#include <libkern/OSAtomic.h>
#include "MusicDeviceBase.h"
#include "LockFreeFIFO.h"
#include "SynthEvent.h"
#include "SynthNote.h"
#include "SynthElement.h"
 
/////////////////////////////////
 
typedef LockFreeFIFOWithFree<SynthEvent> SynthEventQueue;
 
class AUInstrumentBase : public MusicDeviceBase
{
public:
   
   /* =========== 中略 =========== */
 
    virtual OSStatus            Initialize();
    
    /*! @method Parts */
    AUScope &                   Parts() { return mPartScope; }
 
    /*! @method GetPart */
    AUElement *                 GetPart( AudioUnitElement inElement)
    {
        return mPartScope.SafeGetElement(inElement);
    }
 
    virtual AUScope *           GetScopeExtended (AudioUnitScope inScope);
 
    virtual AUElement *         CreateElement(          AudioUnitScope                  inScope,
                                                        AudioUnitElement                inElement);
 
    virtual void                CreateExtendedElements();
 
    virtual void                Cleanup();
    

    /* =========== 中略 =========== */

 
    virtual OSStatus            Render(                 AudioUnitRenderActionFlags &    ioActionFlags,
                                                        const AudioTimeStamp &          inTimeStamp,
                                                        UInt32                          inNumberFrames);

    virtual OSStatus            StartNote(      MusicDeviceInstrumentID     inInstrument, 
                                                MusicDeviceGroupID          inGroupID, 
                                                NoteInstanceID *            outNoteInstanceID, 
                                                UInt32                      inOffsetSampleFrame, 
                                                const MusicDeviceNoteParams &inParams);
 
    virtual OSStatus            StopNote(       MusicDeviceGroupID          inGroupID, 
                                                NoteInstanceID              inNoteInstanceID, 
                                                UInt32                      inOffsetSampleFrame);
   
    /* =========== 中略 =========== */

    
    void                SetNotes(UInt32 inNumNotes, UInt32 inMaxActiveNotes, SynthNote* inNotes, UInt32 inNoteSize);
    
    void                PerformEvents(   const AudioTimeStamp &         inTimeStamp);
    OSStatus            SendPedalEvent(MusicDeviceGroupID inGroupID, UInt32 inEventType, UInt32 inOffsetSampleFrame);
    virtual SynthNote*  VoiceStealing(UInt32 inFrame, bool inKillIt);
    UInt32              MaxActiveNotes() const { return mMaxActiveNotes; }
    UInt32              NumActiveNotes() const { return mNumActiveNotes; }
    void                IncNumActiveNotes() { ++mNumActiveNotes; }
    void                DecNumActiveNotes() { --mNumActiveNotes; }
    UInt32              CountActiveNotes();
    
    SynthPartElement *  GetPartElement (AudioUnitElement inPartElement);
    
            // this call throws if there's no assigned element for the group ID
    virtual SynthGroupElement * GetElForGroupID (MusicDeviceGroupID inGroupID);
    virtual SynthGroupElement * GetElForNoteID (NoteInstanceID inNoteID);
 
    SInt64 mAbsoluteSampleFrame;
 
    
private:
                
    SInt32 mNoteIDCounter;
    
    SynthEventQueue mEventQueue;
    
    UInt32 mNumNotes;
    UInt32 mNumActiveNotes;
    UInt32 mMaxActiveNotes;
    SynthNote* mNotes;  
    SynthNoteList mFreeNotes;
    UInt32 mNoteSize;
    
    AUScope         mPartScope;
    const UInt32    mInitNumPartEls;
};
 
////////////////////////////////////////////////////////////////////////////////////////////////////////////
 
class AUMonotimbralInstrumentBase : public AUInstrumentBase
{
public:
    AUMonotimbralInstrumentBase(
                            AudioComponentInstance          inInstance, 
                            UInt32                          numInputs,
                            UInt32                          numOutputs,
                            UInt32                          numGroups = 16,
                            UInt32                          numParts = 1);
                                        
    virtual OSStatus            RealTimeStartNote(          SynthGroupElement           *inGroup, 
                                                            NoteInstanceID              inNoteInstanceID, 
                                                            UInt32                      inOffsetSampleFrame, 
                                                            const MusicDeviceNoteParams &inParams);
};
 
////////////////////////////////////////////////////////////////////////////////////////////////////////////
 
// this is a work in progress! The mono-timbral one is finished though!
class AUMultitimbralInstrumentBase : public AUInstrumentBase
{
public:
    AUMultitimbralInstrumentBase(
                            AudioComponentInstance          inInstance, 
                            UInt32                          numInputs,
                            UInt32                          numOutputs,
                            UInt32                          numGroups,
                            UInt32                          numParts);
                            
    virtual OSStatus            GetPropertyInfo(        AudioUnitPropertyID             inID,
                                                        AudioUnitScope                  inScope,
                                                        AudioUnitElement                inElement,
                                                        UInt32 &                        outDataSize,
                                                        Boolean &                       outWritable);
 
    virtual OSStatus            GetProperty(            AudioUnitPropertyID             inID,
                                                        AudioUnitScope                  inScope,
                                                        AudioUnitElement                inElement,
                                                        void *                          outData);
 
    virtual OSStatus            SetProperty(            AudioUnitPropertyID             inID,
                                                        AudioUnitScope                  inScope,
                                                        AudioUnitElement                inElement,
                                                        const void *                    inData,
                                                        UInt32                          inDataSize);
 
private:
 
};
 
////////////////////////////////////////////////////////////////////////////////////////////////////////////
 
#endif
```

#### SinSynth.cpp

```cpp
/*
     File: SinSynth.cpp
 Abstract: SinSynth.h
  Version: 1.0.1
 
========= 中略 =========
 
 Copyright (C) 2014 Apple Inc. All Rights Reserved.
 
*/

/* ========= 中略 ========= */

#include "SinSynth.h"
 
static const UInt32 kMaxActiveNotes = 8;

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

const double twopi = 2.0 * 3.14159265358979;

inline double pow5(double x) { double x2 = x*x; return x2*x2*x; }

#pragma mark SinSynth Methods

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

//COMPONENT_ENTRY(SinSynth)
AUDIOCOMPONENT_ENTRY(AUMusicDeviceFactory, SinSynth)

static const AudioUnitParameterID kGlobalVolumeParam = 0;
static const CFStringRef kGlobalVolumeName = CFSTR("global volume");

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
//	SinSynth::SinSynth
//
// This synth has No inputs, One output
//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
SinSynth::SinSynth(AudioUnit inComponentInstance)
	: AUMonotimbralInstrumentBase(inComponentInstance, 0, 1)
{
	CreateElements();
	
	Globals()->UseIndexedParameters(1); // we're only defining one param
	Globals()->SetParameter (kGlobalVolumeParam, 1.0);
}

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
//	SinSynth::~SinSynth
//
//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
SinSynth::~SinSynth()
{
}


/* ========= 中略 ========= */

OSStatus SinSynth::Initialize()
{

/* ========= 中略 ========= */

	AUMonotimbralInstrumentBase::Initialize();

/* ========= 中略 ========= */
	
	return noErr;
}

AUElement* SinSynth::CreateElement(	AudioUnitScope					scope,
									AudioUnitElement				element)
{
	switch (scope)
	{
		case kAudioUnitScope_Group :
			return new SynthGroupElement(this, element, new MidiControls);
		case kAudioUnitScope_Part :
			return new SynthPartElement(this, element);
		default :
			return AUBase::CreateElement(scope, element);
	}
}

OSStatus			SinSynth::GetParameterInfo(		AudioUnitScope					inScope,
														AudioUnitParameterID			inParameterID,
														AudioUnitParameterInfo &		outParameterInfo)
{
	if (inParameterID != kGlobalVolumeParam) return kAudioUnitErr_InvalidParameter;
	if (inScope != kAudioUnitScope_Global) return kAudioUnitErr_InvalidScope;

	outParameterInfo.flags = SetAudioUnitParameterDisplayType (0, kAudioUnitParameterFlag_DisplaySquareRoot);
    outParameterInfo.flags += kAudioUnitParameterFlag_IsWritable;
	outParameterInfo.flags += kAudioUnitParameterFlag_IsReadable;

	AUBase::FillInParameterName (outParameterInfo, kGlobalVolumeName, false);
	outParameterInfo.unit = kAudioUnitParameterUnit_LinearGain;
	outParameterInfo.minValue = 0;
	outParameterInfo.maxValue = 1.0;
	outParameterInfo.defaultValue = 1.0;
	return noErr;
}


/* ========= 中略 ========= */

//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

void			TestNote::Release(UInt32 inFrame)
{
	SynthNote::Release(inFrame);

/* ========= 中略 ========= */

}

void			TestNote::FastRelease(UInt32 inFrame) // voice is being stolen.
{
	SynthNote::Release(inFrame);

/* ========= 中略 ========= */

}

void			TestNote::Kill(UInt32 inFrame) // voice is being stolen.
{
	SynthNote::Kill(inFrame);

/* ========= 中略 ========= */

}

OSStatus		TestNote::Render(UInt64 inAbsoluteSampleFrame, UInt32 inNumFrames, AudioBufferList** inBufferList, UInt32 inOutBusCount)
{
	float *left, *right;

/* ========= 中略 ========= */

	float globalVol = GetGlobalParameter(kGlobalVolumeParam);
	
	// TestNote only writes into the first bus regardless of what is handed to us.
	const int bus0 = 0;
	int numChans = inBufferList[bus0]->mNumberBuffers;
	if (numChans > 2) return -1;
	
	left = (float*)inBufferList[bus0]->mBuffers[0].mData;
	right = numChans == 2 ? (float*)inBufferList[bus0]->mBuffers[1].mData : 0;

	double sampleRate = SampleRate();
	double freq = Frequency() * (twopi/sampleRate);

	
/* ========= 中略 ========= */

	switch (GetState())
	{
		case kNoteState_Attacked :
		case kNoteState_Sostenutoed :
		case kNoteState_ReleasedButSostenutoed :
		case kNoteState_ReleasedButSustained :
			{
				for (UInt32 frame=0; frame<inNumFrames; ++frame)
				{
					if (amp < maxamp) amp += up_slope;
					float out = pow5(sin(phase)) * amp * globalVol;
					phase += freq;
					if (phase > twopi) phase -= twopi;
					left[frame] += out;
					if (right) right[frame] += out;
				}
			}
			break;
			
		case kNoteState_Released :
			{
				UInt32 endFrame = 0xFFFFFFFF;
				for (UInt32 frame=0; frame<inNumFrames; ++frame)
				{
					if (amp > 0.0) amp += dn_slope;
					else if (endFrame == 0xFFFFFFFF) endFrame = frame;
					float out = pow5(sin(phase)) * amp * globalVol;
					phase += freq;
					left[frame] += out;
					if (right) right[frame] += out;
				}
				if (endFrame != 0xFFFFFFFF) {

/* ========= 中略 ========= */

					NoteEnded(endFrame);
				}
			}
			break;
			
		case kNoteState_FastReleased :
			{
				UInt32 endFrame = 0xFFFFFFFF;
				for (UInt32 frame=0; frame<inNumFrames; ++frame)
				{
					if (amp > 0.0) amp += fast_dn_slope;
					else if (endFrame == 0xFFFFFFFF) endFrame = frame;
					float out = pow5(sin(phase)) * amp * globalVol;
					phase += freq;
					left[frame] += out;
					if (right) right[frame] += out;
				}
				if (endFrame != 0xFFFFFFFF) {

/* ========= 中略 ========= */
					
					NoteEnded(endFrame);
				}
			}
			break;
		default :
			break;
	}
	return noErr;
}


```

よく見ると、シンセサイザーとして必要な機能が定義されているのがわかりますね。どうやら、 **Render**という関数で音を生成するようです。

SetNoteなどはMIDIの信号を送信するのでしょうね。

ここで、さらに #include に注目してください。

```cpp
#include <vector>
#include <stdexcept>
#include <AudioUnit/AudioUnit.h>
#include <CoreAudio/CoreAudio.h>
#include <libkern/OSAtomic.h>
#include "MusicDeviceBase.h"
#include "LockFreeFIFO.h"
#include "SynthEvent.h"
#include "SynthNote.h"
#include "SynthElement.h"
```

ここを見ると分かるのですが、AUに必要なクラス群がよく分かります。

- AudioUnit/AudioUnit.h は必須です。これがないと始まりません。
- CoreAudio/CoreAudio.h Macの音声を操る、最も重要なフレームワークです。

つまり、この２つが一番重要だと分かります。

## AUのクラス構成について

- Appendix: Audio Unit Class Hierarchy
https://developer.apple.com/library/mac/documentation/musicaudio/Conceptual/AudioUnitProgrammingGuide/AudioUnitClassHierarchy/AudioUnitClassHierarchy.html

に詳しく解説されています。

<img alt="Hierarchy" src="http://imgur.com/OsvMHgK.jpg" />

テキスト化したものが以下です。

```tex

    |-- AUElementCreator
|<--|
|   |-- ComponentBase
|
|--> AUBase -- AUScope -- AUElement
|                         |
|                         |----- AUOElement
|                         |  |
|                         |  |--- AUInputElement
|                         |  |
|                         |  |--- AUOutputElement
|-- AUOutputBase          |
|                         |--- SynthElement
|                           |
|                           |--- SynthPartElement
|                           |
|                           |--- SynthGroupElement
|                             |--- SynthNoteList ...
|                               |--- SynthNote ...
|
|-- AUEffectBase -- AUKernelBase ...
|    |
|    |   AUMIDIBase
|    |   |  |
|    |   |  | 
|    |   |  --_
|    |---|----- AUMIDIEffectBase
|        |
|        |--_ 
|------------ MusicDeviceBase
               |
               |-- [[AUInstrumentBase]]
                    |
                    |-- AUMonotimbralInstrumentBase
                    |
                    |-- AUMultitimbralIntrumentBase

```

括弧をつけたところが **AUInstrumentBase** です。

なんかよくわかりませんがそういうことにしておきましょう。

とにかく、 **CoreAudio** と **AudioUnit** は重要だということを覚えておいてください。

詳しくは他のサイトや、Apple Developer Networkを活用して下さい。

# まとめ

今回は、VSTとAudioUnitの入門として書きました。いかがでしたでしょうか。

これを参考に、少しでもVSTやAUが増えれば幸いです。

# 参考になる文献

### VST

- VSTの作り方
http://www.g200kg.com/jp/docs/makingvst/

- C++で音作り
http://www39.atwiki.jp/vst_prog/

- VSTプラグインを自作する
http://gyabo.sakura.ne.jp/tips/vst0.html

### AudioUnits

- ダウンロード可能な Audio Unit 関連のサンプルコード11個
http://d.hatena.ne.jp/shu223/20130221/1361394373

- Getting Started With Audio Unit - My Codex Leicester
http://nagano.monalisa-au.org/archives/category/getting-started-with-audio-unit?order=ASC

### 音響生成プログラミング

- C言語で始める音のプログラミング
http://floor13.sakura.ne.jp/book03/book03.html

- モバイルとシンセで音楽作りな話
https://pfjksynth.wordpress.com/2013/03/

- PDコミュニティサイト（音声生成とフィルタの基本がわかる）
http://puredata.info/

- ChucK （リアルタイム音響生成プログラミング言語）
http://chuck.cs.princeton.edu/
