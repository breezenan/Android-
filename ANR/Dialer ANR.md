## 问题描述

拨号盘输入上万位号码，手机出现卡顿，多次点击导致ANR

## 解决

trace log

```
----- pid 16020 at 2019-01-02 09:15:12 -----
Cmd line: com.android.dialer
Build fingerprint: 'TECNO/VP572/TECNO-BB2:8.1.0/O11019/ABC-190825V1:userdebug/release-keys'
ABI: 'arm'
Build type: optimized
Zygote loaded classes=5049 post zygote classes=1451
Intern table: 75840 strong; 129 weak
JNI: CheckJNI is off; globals=559 (plus 28 weak)
suspend all histogram:	Sum: 106.416ms 99% C.I. 12us-237.706us Avg: 61.053us Max: 3376us
DALVIK THREADS (24):
"main" prio=5 tid=1 Runnable
  | group="main" sCount=0 dsCount=0 flags=0 obj=0x72064998 self=0xa66ac000
  | sysTid=16020 nice=-10 cgrp=default sched=0/0 handle=0xaacc84a4
  | state=R schedstat=( 289129601940 467451626 26359 ) utm=25808 stm=3103 core=1 HZ=100
  | stack=0xbe632000-0xbe634000 stackSize=8MB
  | held mutexes= "mutator lock"(shared held)
  at java.util.regex.Matcher.resetForInput(Matcher.java:1093)
  at java.util.regex.Matcher.reset(Matcher.java:1084)
  at java.util.regex.Matcher.reset(Matcher.java:1052)
  at java.util.regex.Matcher.<init>(Matcher.java:180)
  at java.util.regex.Pattern.matcher(Pattern.java:1006)
  at com.android.i18n.phonenumbers.AsYouTypeFormatter.attemptToExtractIdd(AsYouTypeFormatter.java:574)
  at com.android.i18n.phonenumbers.AsYouTypeFormatter.inputDigitWithOptionToRememberPosition(AsYouTypeFormatter.java:334)
  at com.android.i18n.phonenumbers.AsYouTypeFormatter.inputDigit(AsYouTypeFormatter.java:299)
  at android.telephony.PhoneNumberFormattingTextWatcher.getFormattedNumber(PhoneNumberFormattingTextWatcher.java:157)
  at android.telephony.PhoneNumberFormattingTextWatcher.reformat(PhoneNumberFormattingTextWatcher.java:140)
  at android.telephony.PhoneNumberFormattingTextWatcher.afterTextChanged(PhoneNumberFormattingTextWatcher.java:108)
  - locked <0x043bb0e5> (a android.telephony.PhoneNumberFormattingTextWatcher)
  at android.widget.TextView.sendAfterTextChanged(TextView.java:9402)
  at android.widget.TextView$ChangeWatcher.afterTextChanged(TextView.java:11975)
  at android.text.SpannableStringBuilder.sendAfterTextChanged(SpannableStringBuilder.java:1262)
  at android.text.SpannableStringBuilder.replace(SpannableStringBuilder.java:574)
  at android.text.SpannableStringBuilder.replace(SpannableStringBuilder.java:504)
  at android.text.SpannableStringBuilder.replace(SpannableStringBuilder.java:502)
  at android.text.method.NumberKeyListener.onKeyDown(NumberKeyListener.java:131)
  at android.widget.TextView.doKeyDown(TextView.java:7340)
  at android.widget.TextView.onKeyDown(TextView.java:7117)
  at com.android.dialer.app.dialpad.DialpadFragment.keyPressed(DialpadFragment.java:1202)
  at com.android.dialer.app.dialpad.DialpadFragment.onPressed(DialpadFragment.java:1244)
  at com.android.dialer.dialpadview.DialpadKeyButton.setPressed(DialpadKeyButton.java:117)
  at android.view.View.setPressed(View.java:9958)
  at android.view.View.onTouchEvent(View.java:13078)
  at android.view.View.dispatchTouchEvent(View.java:11818)
  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:3016)
  at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2666)
  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:3022)
  at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2680)
  at android.view.ViewGroup.dispatchTransformedTouchEvent(ViewGroup.java:3022)
  at android.view.ViewGroup.dispatchTouchEvent(ViewGroup.java:2680)
```

从`- locked <0x043bb0e5> (a android.telephony.PhoneNumberFormattingTextWatcher`看出阻塞发生在该类中，看该行log上面知道`afterTextChanged`方法中比较耗时。

events_log

```
01-02 09:15:12.217329   617   661 I am_anr  : [0,16020,com.android.dialer,885767749,Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 9.  Wait queue head age: 6629.1ms.)]
```

可以看出触摸事件下发超时，处于等待队列中还有9个事件待处理，从而确定是我们触摸事件发生后执行比较耗时导致

sys_log

```
01-02 09:15:16.905863   617   661 I AnrManager: ANR in com.android.dialer (com.android.dialer/.app.DialtactsActivity), time=23908094
01-02 09:15:16.905863   617   661 I AnrManager: Reason: Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 9.  Wait queue head age: 6629.1ms.)
01-02 09:15:16.905863   617   661 I AnrManager: Load: 9.56 / 9.72 / 9.67
01-02 09:15:16.905863   617   661 I AnrManager: Android time :[2019-01-02 09:15:16.90] [23912.781]
01-02 09:15:16.905863   617   661 I AnrManager: CPU usage from 16042ms to 41ms ago (2019-01-02 09:14:56.171 to 2019-01-02 09:15:12.172):
01-02 09:15:16.905863   617   661 I AnrManager:   126% 16020/com.android.dialer: 111% user + 14% kernel / faults: 156322 minor 1 major
01-02 09:15:16.905863   617   661 I AnrManager:   10% 268/android.hardware.audio@2.0-service-mediatek: 6.9% user + 3.3% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   7.1% 617/system_server: 4.4% user + 2.6% kernel / faults: 1743 minor 15 major
01-02 09:15:16.905863   617   661 I AnrManager:   5.3% 321/audioserver: 3.6% user + 1.6% kernel / faults: 70 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.2% 966/com.google.android.gms.persistent: 0.1% user + 0% kernel / faults: 302 minor 1 major
01-02 09:15:16.905863   617   661 I AnrManager:   0.8% 281/surfaceflinger: 0.4% user + 0.4% kernel / faults: 297 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.5% 275/android.hardware.graphics.composer@2.1-service: 0.2% user + 0.2% kernel / faults: 58 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.6% 243/logd: 0.1% user + 0.4% kernel / faults: 16 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.6% 332/mobile_log_d: 0.3% user + 0.3% kernel / faults: 37 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.4% 277/merged_hal_service: 0.1% user + 0.3% kernel / faults: 20 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 1625/com.google.android.gms: 0% user + 0% kernel / faults: 158 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0.3% 952/com.android.systemui: 0.1% user + 0.1% kernel / faults: 39 minor 2 major
01-02 09:15:16.905863   617   661 I AnrManager:   0.2% 7/rcu_preempt: 0% user + 0.2% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.2% 69/kswapd0: 0% user + 0.2% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 179/bat_thread_kthr: 0% user + 0.1% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 202/f2fs_discard-17: 0% user + 0.1% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 144/mmcqd/0: 0% user + 0.1% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 175/mtk-tpd: 0% user + 0.1% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 320/aal: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0.1% 503/mdlogger: 0% user + 0.1% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 1//init: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 53/cfinteractive: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 54/ion_mm_heap: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 94/hps_main: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 130/frame_update_wo: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 244/servicemanager: 0% user + 0% kernel / faults: 18 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0% 280/lmkd: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 335/wificond: 0% user + 0% kernel / faults: 48 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0% 499/aee_aed: 0% user + 0% kernel / faults: 102 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0% 504/gsm0710muxd: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 616/rilproxy: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 5298/tx_thread: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 5704/com.transsion.hilauncher: 0% user + 0% kernel / faults: 82 minor 127 major
01-02 09:15:16.905863   617   661 I AnrManager:   0% 13116/kworker/u8:2: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 14044/kworker/0:0: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager:   0% 14715/adbd: 0% user + 0% kernel / faults: 238 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0% 15793/com.android.chrome:sandboxed_process1: 0% user + 0% kernel / faults: 188 minor
01-02 09:15:16.905863   617   661 I AnrManager:   0% 15904/com.afmobi.boomplayer: 0% user + 0% kernel / faults: 41 minor 13 major
01-02 09:15:16.905863   617   661 I AnrManager:  +0% 17462/app_process: 0% user + 0% kernel
01-02 09:15:16.905863   617   661 I AnrManager: 49% TOTAL: 40% user + 8.4% kernel + 0% iowait
01-02 09:15:16.906148   617   661 I AnrManager: dumpAnrDebugInfo end: AnrDumpRecord{ Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 9.  Wait queue head age: 6629.1ms.) ProcessRecord{655e36c 16020:com.android.dialer/u0a30} IsCompleted:true IsCancelled:false }, isAsyncDump = false
01-02 09:15:16.907483   617   684 I AnrManager: dumpAnrDebugInfoLocked: AnrDumpRecord{ Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 9.  Wait queue head age: 6629.1ms.) ProcessRecord{655e36c 16020:com.android.dialer/u0a30} IsCompleted:true IsCancelled:false }, isAsyncDump = true
01-02 09:15:16.907715   617   684 I AnrManager: dumpAnrDebugInfo end: AnrDumpRecord{ Input dispatching timed out (Waiting to send non-key event because the touched window has not finished processing certain input events that were delivered to it over 500.0ms ago.  Wait queue length: 9.  Wait queue head age: 6629.1ms.) ProcessRecord{655e36c 16020:com.android.dialer/u0a30} IsCompleted:true IsCancelled:false }, isAsyncDump = true
```

从sys_log中`I AnrManager:   126% 16020/com.android.dialer: 111% user + 14% kernel / faults: 156322 minor 1 major`可以看出是Dialer应用本身比较耗cpu。



经过以上分析，可知主要还是触摸事件无法在一定时间内响应，导致ANR，具体是由于PhoneNumberFormattingTextWatcher类`afterTextChanged`的方法耗时，对于系统API，一般不会存在性能问题，但我们需要注意的是，当Dialer输入字符特别多时才会执行时间长，为了验证该方法具体执行时间，写如下示例：

```java
System.out.println(System.currentTimeMillis());
PhoneNumberFormattingTextWatcher textWatcher = new PhoneNumberFormattingTextWatcher();
CharSequence text = new SpannableStringBuilder("很长很长的字符串");
textWatcher.afterTextChanged((Editable) text);
System.out.println(System.currentTimeMillis());
```

经测试得知，随着SpannableStringBuilder中字符的长度增加，执行时间直线增加，在存在几千字符时，执行时间已经达到2-3s，现在问题清楚了，就是输入字符过多导致卡顿，进而导致ANR。

**解决办法**，经过与NOKIA 6(Android 9.0) Dialer 对比，其是通过限制输入字符1000为解决办法，而且电话号码有那么长的吗？故最终解决办法就是如此：

```java
editText.setFilters(new InputFilter[]{new InputFilter.LengthFilter(1000)});
```

这样当字符达到1000时就无法再输入了，但是仍然是可编辑的。