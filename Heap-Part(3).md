# Arena And Subheaps

Large memory allocation တွေလုပ်တဲ့ အခါ  malloc အနေနဲ့
normal heap path တွေ ဖြစ်တဲ့ reuse,top chuckနဲ့ extend တွေကို By Pass Skip လုပ်ပစ်ပြီး mmap နဲ့ OS memory ကနေ တစ်ခါထဲနောက် heap တစ်ခုတည်ဆောက်လိုက်ပါတယ်။

ဟို အရင် bin တွေသုံးတာမျိုးနဲ့ မတူတာက အဲ့ chuck ကို free(ptr) လုပ်လိုက်တဲ့ အခါ region တစ်ခုလုံးကို OS ကို ပြန်ပေးလိုက်ရပြီး အထဲမှာ ရှိတဲ့ Metadata တွေ Userdata တွေကို access ပြန်လုပ်လို့ Reuse လုပ်လို့မရတော့ပါဘူး။

ဒီအကြောင်းတွေ ဘာလို့ပြောရလည်းဆိုရင် Threshold ဆိုတဲ့ ကောင်ကြောင့်ပါ ။ ဒီကောင်က Average အနေနဲ့

    32 bit တွေမှာဆိုရင် 128 to 512 KB ထိရှိပြီး ၊ 64 bit မှာတော့ 32MB ထိရှိပါတယ်။

Threshold နည်းနေရင် ဥပမာ request လိုက်တာက 100KB threshold က 125KB ဆိုရင် ဒါဟာ Normal Heap Path ကနေဖြတ်သန်းနေတာ ကို သိနိုင်ပါတယ်။

threshold များနေရင်တော့ ဥပမာ 200KB request လိုက်တယ် threshold က 125 KB ဆိုရင် သေချာပါတယ်  ဒီကောင်ကြီးက mmap နဲ့ ခွဲထုတ်နေ ရပြီဆိုတာ။

mmap နဲ့ ခွဲထုတ်ထားတဲ့ chuck တွေကို subheap တွေလိုခေါ်ပြီး ဒီကောင်တွေက OS ကနေလာပြီး free ရင် OS ကိုပြန်ရ‌တာကြောင့် , vuln တွေ့ဖို့ဆိုတာ ခပ်ရှားရှားရယ်ပါ။
mmap နဲ့ ခွဲထုတ်ထားတဲ့ heap တွေကို Secondry arena လို့ခေါ်ပြီး First Arena ကတော့ main heap ပဲဖြစ်ပါတယ်။)

## Subheaps

subheap တွေဆို တာ အပိုထပ် ခွဲထားတဲ့ arena မှာ ရှိတဲ့ heap တွေကိုပြောတာဖြစ်ပြီး ၊ အဲ့ subheap တွေထဲမှာ chuck တွေလည်း ပါဝင်နေမှာပါ။

First Arena ဖြစ်တဲ့ Main Heap ထဲမှာဆိုရင် Memory extend ဖို့ဆို sbrk သုံးပါတယ်။

Secondry Arena ရဖို့ဆိုရင်တော့ malloc က New Arena တစ်ခု ခွဲပေးမှရမှာပါ။

ဒါကြောင့် Secondry Arena မှာ စပြီး Subheap တွေ စလုပ်တဲ့ အခါ mmap နဲ့ mprotect ကိုသုံးရပါတယ်။
ဘာလို့ဆို Subheap တွေစပြီး တည်ဆောက်တဲ့ အခါ reserve address တွေကို အဲ့ subheap တွေအတွက် အရင်ပေးထားလိုက်ပါတယ်။
ပြီးရင်တော့ အဲ့ address တွေကို PROT_NONE အဖြစ်သတ်မှတ်ထားပေးပါတယ်။
  
    PROT_NONE ဆိုတာ kernal ကို ဒီ address တွေ သုံးလို့မရသေးဘူး ဆိုတဲ့ ပုံစံ အကြောင်းကြားထားတာပါ။

အဲ့ reserve address တွေ heap တစ်ခုလုံးကို ပေးပြီးရင်တော့ mprotect ဆိုတဲ့ကောင်ကနေ NONE ကနေ READ , WRITE flag တွေ change ပေးခြင်းအားဖြင့် physical address တွေကို repeat လုပ်ပြီး ပေး‌နိုင်ပါပြီ။

ဒါဆိုရင်တော့ subheap တွေက fully grow ဖြစ်သွားပါပြီ။

အကယ်၍ program တစ်ခုက 2 threads နဲ့လာခဲ့ရင် 
main heap ကနေ single thread တစ်ခုကိုင်ပါတယ် ၊ ကျန်တဲ့ thread ကိုတော့ new arena ( second arena) က ကိုင်ပါတယ်။ သုံးခုဆိုရင် နောက်ထပ် ‌arena တစ်ခုပေါ့။

Why multithreading is important in next part.
