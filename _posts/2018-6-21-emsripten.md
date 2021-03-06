---
layout: post
title: مقدمة   تحويل LLVM Bitcode إلى شيفرة JavaScript بكفاءة 
---

قد يبدوا العنوان غريبا بعض الشيء ، ولكني أصبو من خلاله إلى صيد العديد من المنافع وإعطاءها لك ، في هذه المقالة سنتعرف على Emscripten مفتوحة المصدر ، وما يمكنك فعله من خلالها  ، كما سنتطرق إلى آلية تحويل أي كود مكتوب بلغة c++ إلى كود جافاسكريبت يمكن تشغيله على أي متصفح أو أي مستعرص node.js.

**لاحظ أن شيفرة c++ ينتج عنها LLVM أو Low Level Virtual Machine**

### مقدمة : لماذا جافاسكريبت ؟
جافاسكريبت JS  هي اللغة البرمجية الوحيدة التي يمكن تنفيذ شيفرتها على أي مستعرض ويب في أي نظام تشغيل ، لا يهم نوع المتصفح الذي تستخدمه ، فهو بالتأكيد قادر على ترجمة شيفرة جافاسكريبت ، هذا اﻷمر يجعل من حافاسكريبت لغة مرنة للغاية مع قدرة تفسيرية عالية .
لذا تعد جافاسكريبت الخيار اﻷول للمطورين الذين يسعون إلى تطوير تطبيقات موحدة تعمل على منصات تشغيل مختلفة ، فبالتأكيد هاتفك الذكي  يحتوي على مستعرض ويب ، وربما ساعتك الذكية كذلك ، وقد نمتد قليلا الى  مختلف معداتك اليومية ،  لهذا السبب ضهرت أطر عمل مختلفة لتساعد هؤلاء المطورين على إتمام تطوير تطبيقاتهم بفعالية أعلى عبر جافاسكريبت مثل إطار electron  الذي يمكنك من تطوير تطبيقات سطح مكتب بإستخدام  لغات الويب المختلفة.

### الفجوة
على الرغم من ذلك ، لاتزال لغات برمجة أخرى مثل c++ ذات قدرة عالية على تطوير تطبيقات أصلية ضمن منصاتها ، والعديد من التطبيقات والألعاب الهامة والفريدة من نوعها تم كتابتها عبر هذه اللغة ، المشكلة تكمن أن c++ وعلى خلاف JS لن تجد المفسر  الخاص بها في كل مكان ،وستعمل ضمن بيئة محدودة من نظم التشغيل ، وربما قد تحتاج إلى إعادة كتابة برامجك بلغة أخرى لتشغيلها ضمن بيئة أخرى ، مثل الويب .
لذا سيكون من الجيد لو أستطعنا تحويل الشيفرة البرمجية القوية التي تم كتابتها بإحدى اللغات مثل c++  إلى شيفرة برمجية بنفس الكفاءة ولكن مكتوبة بلغة JS  ويمكن تشغيلها على أنماط واسعة من بيئات العمل .

ربما نلاحظ هنا أننا أمام عدد واسع من الخيارات والإحتمالات  الإيجابية ، يمكن من خلال Ems. أن يستفيدي مطوري c++ بتحويل مشاريعهم وبرمجياتهم إلى JS  وتشغيلها على بيئات الويب دون أدنى معرفة بها  ، ويمكن لمطوري الويب الإستفادة من البرمجيات مفتوحة المصدر المكتوبة بلغة C++ لتشغيلها أو تضمينها ضمن مشاريعهم المختلفة.

من خلال Ems. ستتمكن من :

* ترجمة شيفرة مكتوبة بلغة c++ إلى شيفرة مكتوبة بلغة JS
* ترجمة أي لغة برمحة يمكن تحويل شفرتها إلى LLVM إلى شفرة JS ذات كفاءة الية

هذه الإمكانات تم ذكرها في موقع Ems .

لذا يمكننا الوصول في نهاية المطاف إلى استنتاج مفاده  أن Emscripten  **مترجم  يعمل على تحويل الـ LLVM إلى شفرة مصدرية مكتوبة بلغة جافاسكريبت**  .
واحدة من التطبيقات المفضلة التي تم تحويلها إلى JS عبر Ems.  هي مكتبة الرؤية الحاسوبية المميزة openCV  والتي صدرت نسخة منها تحت إسم openCV.Js  مما يعني إمكانية إنشاء تطبيقات للرؤية الحاسوبية من خلال هذه المكتبة بعد أن كانت مقتصرة فقط على c++  و  python  ، أيضا نلاحظ أن webGL  مبنية على openGL  التي كتبت في الأساس بإستخدام c++  ، ربما يمكنك ملاحظة مدى عملية Ems.  . 


### مثالنا اﻷول
في هذه النقطة سنتطرق إلى كيفية تنصيب Ems. على حاسبك ، ثم سنحول شيفرة برمجية بسيطة مكتوبة بلغة c++  إلى شيفرة Js تعمل على كافة المتصفحات .
لتحميل Emscripten SDK (emsdk) Ems. على حاسبك ستحتاج إلى تثبيت بايثون ومن خلال سطر الأوامر نفذ التالي 

```
# Get the emsdk repo
git clone https://github.com/juj/emsdk.git

# Enter that directory
cd emsdk
```
الآن نفذ الأوامر التالية لتحميل الأدوات اللازمة من github وتفعيل emSDK 
```
# Fetch the latest registry of available tools.
./emsdk update

# Download and install the latest SDK tools.
./emsdk install latest

# Make the "latest" SDK "active" for the current user. (writes ~/.emscripten file)
./emsdk activate latest

# Activate PATH and other environment variables in the current terminal
source ./emsdk_env.sh
```

الموقع الرسمي للأداة يذكر مجموعة من الملاحظات حول عملية التنصيب لكل نظام تشغيل ، للإستزادة سأترك الروابط اللازمة أسفل هذه التدوينة .
دعونا الآن نتأكد من أن عملية التنصيب قد تمت ، لنجرب تحويل هذه السيفرة المكتوب بلغة c++ إلى شيفرة JS
```
#include <stdio.h>

int main() {
  printf("hello, world!\n");
  return 0;
}
```
قم بتفعيل Ems عبر تنفيذ الأمر **./emcc -v**  ، هنا يفترض أن لا تحصل أي أخطاء او تظهر أي نوع من التحذيرات  ،لنفترض أن إسم ملف c++ الذي نريد تحويله هو first.c  ، لتبدأ عملية التحويل كل ما عليك فعله هو كتابة الأمر  
```
./emcc tests/first.c
```
تهانينا ! ، من المفترض الآن أن تشاهد ملفين جدد بجانب ملفنا الأصلي الأول  تحت إسم  **node a.out.js** والآخر تحت إسم  **a.out.wasm** ما يهمنا حاليًا هو الملف الأول ، وهو ناتج عملية التحويل إلى جافاسكريبت ، ويقوم بطباعة كلمة Hello World في  الـ console ! 
إستخدم node.Js لتجربة النتيجة النهائية 
>Node.js is a cross-platform runtime environment for server-side and networking applications written in JavaScript. Essentially it allows you to run JavaScript applications outside of a browser context.

خلال اﻷيام القادمة سننظر في البنية الخاصة بـ Ems . وآلية عملها  ، مما يعطينا فهم أعمق لكيفية تحويل LLVM bitcode إلى JS .
 نلتقي لاحثًا  !
