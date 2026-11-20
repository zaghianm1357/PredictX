# PX-DATA-001 — مدل داده بازار و کیفیت داده

- شناسه: `PX-DATA-001`
- نسخه سند: 0.1
- وضعیت: پیشنهاد برای بازبینی
- تاریخ: 2026-11-20
- وابستگی‌ها: PX-RS-001، PX-ARCH-001

## 1. هدف و اصول

این سند قرارداد منطقی داده را برای ورود، استانداردسازی، کنترل کیفیت و اتصال داده بازار به آزمایش‌های PredictX تعریف می‌کند. این سند هنوز انتخاب قطعی vendor، پایگاه داده یا runtime نیست.

اصول:
1. داده خام منبع حذف یا overwrite نمی‌شود.
2. هر رکورد استانداردشده به منبع و نسخه تبدیل قابل ردیابی است.
3. timestamp بدون timezone معتبر به UTC تبدیل نمی‌شود.
4. قیمت، حجم و زمان با معنای واحد و صریح ذخیره می‌شوند.
5. خطاهای کیفیت پنهان یا با حدس پر نمی‌شوند.
6. dataset منتشرشده immutable است؛ اصلاح داده نسخه جدید می‌سازد.
7. شناسه نماد، نماد نمایشی و منبع قیمت سه مفهوم جدا هستند.

## 2. لایه‌های داده

### 2.1 Raw layer

داده ورودی به شکل اصلی ارائه‌دهنده نگهداری می‌شود:
- فایل/پاسخ خام یا snapshot اصلی، بدون تغییر محتوا
- نام منبع و شناسه حساب/محصول داده در صورت مجاز بودن
- زمان دریافت UTC سیستم، جدا از زمان بازار
- شناسه فایل/بچ، checksum و اندازه
- نسخه parser و پارامترهای import
- مجوز، محدودیت نگهداری و طبقه‌بندی محرمانگی

اگر منبع امکان دریافت مجدد داده اصلاح‌شده را دارد، snapshot جدید ثبت می‌شود؛ snapshot قدیمی حذف نمی‌شود.

### 2.2 Canonical layer

داده به قرارداد مشترک PredictX تبدیل می‌شود. نگاشت منبع به canonical باید قابل ردیابی باشد. رکوردی که زمان آن قابل تفسیر نیست می‌تواند در raw باقی بماند، اما تا تعیین تکلیف به مجموعه قابل تحلیل زمانی وارد نمی‌شود.

### 2.3 Research layer

ویژگی‌ها، occurrenceها، outcomeها، آزمایش‌ها و گزارش‌ها با شناسه و نسخه dataset canonical ارتباط می‌گیرند. این لایه نباید داده خام یا نسخه canonical را تغییر دهد.

## 3. هویت ابزار مالی و نماد

نمایش `XAUUSD` به‌تنهایی شناسه یکتای ابزار نیست. قیمت نقدی، CFD و قراردادهای ارائه‌دهندگان مختلف می‌توانند با همین نماد نمایش داده شوند ولی شرایط و قیمت متفاوت داشته باشند.

مدل مفهومی:
- `instrument_id`: شناسه پایدار داخلی PredictX
- `canonical_symbol`: نماد نرمال‌شده نمایشی، مانند XAUUSD
- `asset_class`: مانند FX، metal spot، CFD یا futures؛ فقط در صورت معلوم بودن
- `base_asset` و `quote_asset`: در صورت قابل اعمال بودن
- `provider_id` و `provider_symbol`: منبع و نماد دقیق ارائه‌دهنده
- `instrument_type`، `contract_specification` و `price_unit`: در صورت نیاز و دسترسی
- `valid_from` / `valid_to`: بازه اعتبار مشخصات، اگر مشخصات در زمان تغییر کند

**قاعده:** داده‌های دو provider یا دو نوع قرارداد فقط به دلیل یکسان بودن رشته نماد در یک سری زمانی ادغام نمی‌شوند. ادغام نیازمند تصمیم مستند درباره قابلیت مقایسه و اختلاف قیمت است.

## 4. قرارداد کندل Canonical

هر ردیف کندل باید حداقل این مفاهیم را داشته باشد:

| فیلد | نوع منطقی | الزام | توضیح |
|---|---|---|---|
| instrument_id | string/UUID | الزامی | هویت ابزار، نه صرفاً نماد |
| timeframe | enum/string | الزامی | مانند 1m، 5m، 1h؛ قرارداد دقیق در metadata |
| bar_open_time_utc | timestamp UTC | مشروط | فقط اگر زمان منبع قابل تبدیل معتبر باشد |
| source_timestamp_raw | string/integer | الزامی | مقدار timestamp دقیقاً مطابق منبع |
| source_timestamp_semantics | enum | الزامی | OPEN_TIME، CLOSE_TIME یا UNKNOWN |
| source_timezone | string/enum | الزامی | IANA zone، offset یا UNKNOWN |
| open, high, low, close | decimal | الزامی برای کندل معتبر | بدون گردکردن بی‌ردپا |
| volume_value | decimal/null | اختیاری | مقدار خام حجم در صورت ارائه |
| volume_kind | enum | الزامی | TICK، BASE، QUOTE، CONTRACT، UNKNOWN، NOT_PROVIDED |
| source_id | string | الزامی | شناسه منبع |
| source_record_id | string/null | توصیه‌شده | شناسه رکورد در منبع |
| received_at_utc | timestamp UTC | الزامی | زمان دریافت |
| raw_batch_id | string | الزامی | ارجاع به بچ خام |
| normalization_version | string | الزامی | نسخه قانون تبدیل |
| quality_status | enum | الزامی | ACCEPTED، WARNING، QUARANTINED |

فیلدهای مشتق‌شده مانند `bar_close_time_utc` فقط با قرارداد timeframe و semantics مشخص محاسبه می‌شوند؛ نباید از timestamp مبهم حدس زده شوند.

### 4.1 قرارداد قیمت

- قیمت‌ها با نوع decimal دقیق یا نمایش متنی decimal در interchange نگهداری شوند؛ binary floating-point مبنای ذخیره‌سازی حقیقت قیمت نباشد.
- تعداد رقم اعشار یا tick size از metadata ابزار/منبع تعیین شود، نه با فرض ثابت برای XAUUSD.
- مقدار خام قیمت و مقدار canonical در صورت تغییر قالب یا scale قابل ردیابی باشند.
- قاعده گردکردن، در صورت اجبار فنی، نسخه‌دار و مستند باشد.
- کنترل منطقی پایه: `high >= max(open, close)`، `low <= min(open, close)` و `high >= low`.

### 4.2 قرارداد حجم

حجم بدون معنای واحد قابل مقایسه نیست. مقدار volume باید همراه volume_kind ذخیره شود. حجم tick، حجم پایه، حجم quote و تعداد قرارداد قابل جایگزینی نیستند. اگر نوع حجم معلوم نیست، `UNKNOWN` ثبت می‌شود و تحلیل حجمی باید آن را از تحلیل با حجم معتبر جدا کند.

### 4.3 قرارداد تایم‌فریم و مرز کندل

تعریف timeframe شامل مدت/تقویم، مبنای مرزبندی و semantics timestamp است. برای timeframeهای تقویمی مانند روزانه/هفتگی، مدت ثابت ثانیه‌ای ممکن است کافی نباشد.

در نسخه نخست:
- شناسه کندل بر اساس instrument، timeframe و زمان بازشدن canonical ساخته می‌شود، فقط وقتی زمان بازشدن قابل اثبات باشد.
- کندل‌های intraday بر اساس قرارداد مرزبندی منبع تفسیر می‌شوند.
- timestamp بسته‌شدن کندل در منبعی که زمان پایان را ارسال می‌کند باید به open time تبدیل شود، فقط اگر قاعده timeframe و semantics مستند باشند.
- gap ناشی از تعطیلی بازار را بدون تقویم معاملاتی معتبر، به‌عنوان داده مفقود قطعی برچسب نمی‌زنیم.

## 5. زمان، timezone و DST

برای هر منبع این metadata ثبت شود:
- source_timezone: نام IANA مانند `Europe/London`، offset صریح مانند `+02:00` یا `UNKNOWN`
- timestamp_semantics: زمان بازشدن، بسته‌شدن یا نامعلوم
- timestamp_precision: ثانیه، میلی‌ثانیه، میکروثانیه و غیره
- session/calendar convention در صورت وجود
- زمان دریافت سیستم به UTC

قواعد:
1. زمان خام هرگز بازنویسی نمی‌شود.
2. تبدیل به UTC فقط با offset یا timezone معتبر و قاعده روشن انجام می‌شود.
3. زمان محلی مبهم در تغییر ساعت تابستانی (DST) بدون offset/اطلاعات رفع ابهام، quarantine می‌شود.
4. زمان غیرممکن یا خارج از دامنه قابل پشتیبانی، خطای کیفیت است.
5. زمان بازار و received_at دو محور مستقل‌اند؛ برای داده دیررس یا اصلاح‌شده هر دو نگهداری می‌شوند.
6. اگر timezone نامعلوم است، `UNKNOWN` باقی می‌ماند؛ استفاده از آن برای تحلیل ساعت روز/جلسه یا ترتیب زمانی بین منابع ممنوع است تا زمان حل ابهام.

## 6. کلیدها و یکتایی

### 6.1 هویت کندل

کلید طبیعی پیشنهادی:
`(instrument_id, timeframe, bar_open_time_utc)`

این کلید فقط پس از canonicalization معتبر و تعیین قطعی open time قابل استفاده است. در سطح raw، کلید باید شامل `raw_batch_id` و شماره/شناسه رکورد منبع باشد.

### 6.2 تکراری‌ها

- byte-identical duplicate در یک batch قابل شناسایی و گزارش است.
- duplicate با timestamp یکسان ولی OHLC متفاوت، تعارض است و نباید با آخرین ردیف یا میانگین حل شود.
- duplicate ناشی از دریافت دوباره، با دریافت اصلاحیه provider یکسان فرض نمی‌شود.
- سیاست انتخاب نسخه authoritative باید به source policy وابسته و نسخه‌دار باشد.
- تمام حذف‌ها/ادغام‌ها در QualityReport با تعداد و علت ثبت می‌شوند.

## 7. Dataset و نسخه‌بندی

هر DatasetVersion یک snapshot منطقی تغییرناپذیر است.

### DatasetManifest — حداقل metadata

| فیلد | توضیح |
|---|---|
| dataset_id | شناسه پایدار خانواده dataset |
| dataset_version | نسخه immutable |
| created_at_utc | زمان ساخت manifest |
| source_id / provider | منبع داده |
| instrument_id / provider_symbol | هویت ابزار و نماد منبع |
| timeframe | تایم‌فریم و قرارداد آن |
| time_range_start / end | بازه canonical، در صورت معتبر بودن |
| raw_batch_ids | فهرست ورودی‌های خام |
| content_hash | hash محتوای canonical یا manifest محتوایی |
| schema_version | نسخه schema |
| normalization_version | نسخه تبدیل |
| quality_report_id | گزارش کیفیت مرتبط |
| row_count | تعداد ردیف |
| accepted_count / quarantined_count | شمارش وضعیت‌ها |
| license_reference | ارجاع به مجوز/شرایط استفاده |
| supersedes_dataset_version | نسخه قبلی، در صورت اصلاح |

Hash باید بر مبنای serialization canonical و ترتیب قطعی محاسبه شود؛ hash فایل خام به‌تنهایی معادل hash محتوای canonical نیست. الگوریتم hash و روش serialization در پیاده‌سازی ثبت می‌شوند.

DatasetVersion منتشرشده و استفاده‌شده در Experiment تغییر نمی‌کند. اصلاح provider، backfill، تصحیح parser یا تغییر کیفیت به نسخه جدید منجر می‌شود.

## 8. کنترل کیفیت و وضعیت‌ها

### 8.1 طبقه‌بندی آزمون‌ها

**ساختاری**
- ستون‌های لازم و نوع داده
- مقادیر null در فیلدهای الزامی
- parse شدن timestamp و decimal
- schema و encoding

**زمانی**
- ترتیب زمانی
- timestamp تکراری
- کندل خارج از ترتیب
- فاصله زمانی غیرمنتظره با لحاظ تقویم/session
- DST و timezone مبهم
- دیررس بودن یا بازنویسی داده

**قیمت**
- قیود OHLC
- قیمت صفر/منفی در جایی که نامعتبر است
- جهش یا تغییر غیرعادی برای بررسی؛ نه حذف خودکار
- انطباق scale/tick با مشخصات ابزار، اگر موجود باشد

**حجم**
- مقدار منفی
- نوع حجم نامعلوم
- تغییر ناگهانی واحد یا مقیاس

**پوشش**
- تعداد مورد انتظار در برابر مشاهده‌شده، فقط با تقویم معتبر
- gapهای احتمالی
- پوشش ابتدا/انتهای بازه

### 8.2 وضعیت رکورد و Dataset

- `ACCEPTED`: قواعد سخت‌گیرانه پاس شده‌اند.
- `WARNING`: قابل استفاده با هشدار مشخص و سیاست مصرف صریح.
- `QUARANTINED`: برای محاسبات canonical/زمانی استفاده نمی‌شود تا تعیین تکلیف شود.

وضعیت Dataset:
- `DRAFT`: در حال ورود/تبدیل
- `VALIDATED`: بررسی کیفیت انجام شده
- `PUBLISHED`: snapshot immutable و قابل ارجاع در آزمایش
- `REJECTED`: به دلیل نقض قواعد پذیرش قابل انتشار نیست

حد آستانه پذیرش (مثلاً درصد gap یا تعداد null) به‌صورت ثابت و جهان‌شمول تعیین نمی‌شود؛ باید با نوع منبع، timeframe و هدف پژوهش تنظیم و در QualityPolicy ثبت شود.

### 8.3 QualityIssue و QualityReport

هر Issue حداقل شامل:
- issue_id، dataset_version یا raw_batch_id
- rule_id و rule_version
- severity: INFO، WARNING، ERROR
- affected_record_id/time range
- observed_value و expected_condition، در صورت امکان
- action: NONE، QUARANTINE، REJECT، MANUAL_REVIEW
- reason و timestamp اجرای کنترل

گزارش کیفیت شامل نسخه policy، شمارش هر دسته، نمونه خطاها، بازه‌های متاثر، تصمیم انتشار و شناسه اجراست.

## 9. مدل پژوهشی متصل به Dataset

این سند جزئیات کامل schema آزمایش را جایگزین PX-ARCH-001 نمی‌کند، اما ارتباطات لازم را تثبیت می‌کند:

- **FeatureSet:** dataset_version، feature_definition_version، calculation_version و زمان دسترسی هر مقدار.
- **BehaviorOccurrence:** behavior_definition_version، dataset_version، instrument/timeframe، decision_time و reference candle id.
- **BehaviorOutcome:** occurrence_id، outcome_protocol_version، horizon، start/end reference و status.
- **Experiment:** dataset_version، code_commit، protocol_version، پارامترها، splitهای زمانی و خروجی validation.
- **QualityReport:** dataset_version یا raw_batch_id و quality_policy_version.

هیچ occurrence یا outcome نباید صرفاً با نماد و تاریخ به داده متصل شود؛ ارجاع باید به نسخه و شناسه پایدار داده باشد.

## 10. قالب ذخیره‌سازی — پیشنهاد اولیه

تا زمان benchmark و شناخت منبع، این انتخاب‌ها پیشنهادند، نه تعهد نهایی:

1. **Raw:** نگهداری snapshot در قالب اصلی منبع، همراه checksum و manifest؛ فایل خام در Git قرار نمی‌گیرد.
2. **Canonical analytical data:** قالب ستونی و فشرده مانند Parquet گزینه پیش‌فرض بررسی است؛ برای scanهای زمانی و تحلیل batch مناسب است. partitioning باید پس از شناخت حجم و الگوی query تعیین شود.
3. **Metadata و registry:** در MVP می‌توان metadata را با manifestهای نسخه‌دار و یک catalog سبک مدیریت کرد. انتخاب SQLite یا پایگاه داده server-based به نیاز هم‌زمانی، حجم metadata و استقرار وابسته است.
4. **Research artifacts:** خروجی آزمایش جدا از داده اصلی، با ارجاع به experiment_id و dataset_version.

CSV برای تبادل و نمونه‌گیری مناسب است، اما به‌تنهایی قرارداد دقیق نوع داده، schema evolution و provenance را تضمین نمی‌کند. استفاده از آن به‌عنوان تنها منبع حقیقت توصیه نمی‌شود.

## 11. سیاست اصلاح و backfill

- داده جدید یا اصلاحی در batch مستقل وارد می‌شود.
- رکوردهای متعارض با شناسه و نسخه منبع مقایسه می‌شوند.
- قانون precedence بین provider revisions باید مستند باشد.
- backfill ممکن است برچسب‌ها/ویژگی‌ها/outcomeهای بعدی را تغییر دهد؛ بنابراین DatasetVersion و Experiment جدید لازم است.
- نتایج آزمایش قبلی به dataset جدید منتقل یا overwrite نمی‌شوند.
- تفاوت نسخه‌ها با گزارش delta (افزوده/حذف/اصلاح‌شده) قابل بررسی است.

## 12. حداقل schemaهای مفهومی

| موجودیت | کلید | ارتباط مهم |
|---|---|---|
| DataSource | source_id | یک منبع چند batch دارد |
| RawBatch | raw_batch_id | به source و checksum متصل است |
| Instrument | instrument_id | مشخصات هویت ابزار |
| Dataset | dataset_id | خانواده نسخه‌ها |
| DatasetVersion | dataset_version_id | به manifest و QualityReport متصل |
| Candle | candle_id یا کلید طبیعی | عضو یک DatasetVersion |
| QualityPolicy | policy_version | قواعد کنترل |
| QualityReport | quality_report_id | برای batch یا dataset |
| FeatureDefinition | feature_definition_version | تعریف تغییرناپذیر |
| FeatureSet | feature_set_id | به dataset version متصل |
| BehaviorDefinition | behavior_definition_version | طبق ADR-0001 |
| BehaviorOccurrence | occurrence_id | به dataset و definition متصل |
| Outcome | outcome_id | به occurrence و protocol متصل |
| Experiment | experiment_id | تمام نسخه‌ها و splitها را ثبت می‌کند |

نام و نوع فیلدهای نهایی، enumها، serialization و migration در زمان پیاده‌سازی schema تعیین می‌شوند.

## 13. امنیت و مجوز

- توکن، credential، شناسه حساب حساس و داده دارای محدودیت در Git ثبت نمی‌شود.
- مجوز vendor، بازتوزیع، ذخیره‌سازی تاریخی و استفاده پژوهشی باید پیش از ingest بررسی شود.
- دسترسی raw و artifact باید متناسب با مجوز منبع باشد.
- logها فقط شناسه‌های غیرحساس و metadata عملیاتی را ثبت می‌کنند.
- retention و حذف قانونی باید با نسخه‌پذیری پژوهش سازگار و مستند باشد.

## 14. معیار پذیرش PX-DATA-001

- [ ] raw snapshot با checksum و source metadata قابل ردیابی باشد.
- [ ] canonical candle قرارداد زمان، OHLC و volume_kind صریح داشته باشد.
- [ ] timestamp با timezone نامعلوم به UTC تبدیل نشود.
- [ ] اختلاف duplicateها به‌جای حذف خاموش، گزارش شود.
- [ ] quality policy و گزارش با نسخه ثبت شوند.
- [ ] DatasetVersion منتشرشده immutable باشد.
- [ ] هر feature/occurrence/outcome به نسخه داده ارجاع دهد.
- [ ] آزمون fixture برای gap، duplicate، DST، داده نامرتب، OHLC نامعتبر و حجم نامعلوم وجود داشته باشد.
- [ ] هیچ داده واقعی بدون بررسی مجوز منبع وارد مخزن Git نشود.

## 15. موارد باز برای تصمیم پیش از پیاده‌سازی

1. منبع/ارائه‌دهنده داده و مجوز استفاده
2. Spot، CFD یا instrument دقیق مورد تحلیل
3. قرارداد timestamp و timezone منبع
4. تایم‌فریم‌های MVP و تقویم معاملاتی
5. وجود و معنای volume
6. حجم تاریخی و نرخ به‌روزرسانی
7. قالب raw واقعی و مقصد ذخیره‌سازی
8. نیاز به catalog محلی یا پایگاه داده چندکاربره
9. سیاست backfill و اصلاح داده provider

این موارد باید با نمونه واقعی داده و شرایط مجوز پاسخ داده شوند؛ پیش از آن، هیچ ادعایی درباره پوشش کامل یا کیفیت یک منبع خاص مطرح نمی‌شود.
