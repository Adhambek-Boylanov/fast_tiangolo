# FastAPI Loyihasi - Backend

## Talablar

* [Docker](https://www.docker.com/).
* [uv](https://docs.astral.sh/uv/) — Python paketlari va muhitini boshqarish uchun.

## Docker Compose

Lokal development muhitini ishga tushirish uchun [../development.md](../development.md) faylidagi qo'llanmaga amal qiling.

## Umumiy ish jarayoni

Standart holatda bog'liqliklar (dependencies) [uv](https://docs.astral.sh/uv/) yordamida boshqariladi, uni o'rnatib oling.

`./backend/` katalogidan barcha bog'liqliklarni quyidagicha o'rnatishingiz mumkin:

```console
$ uv sync
```

Keyin virtual muhitni faollashtiring:

```console
$ source .venv/bin/activate
```

Muharriringiz to'g'ri Python virtual muhitidan foydalanayotganiga ishonch hosil qiling — interpretator manzili: `backend/.venv/bin/python`.

Ma'lumotlar va SQL jadvallari uchun SQLModel modellarini `./backend/app/models.py` faylida, API endpointlarni `./backend/app/api/` katalogida, CRUD (Create, Read, Update, Delete) yordamchi funksiyalarini `./backend/app/crud.py` faylida o'zgartiring yoki qo'shing.

## VS Code

Backendni VS Code debugger orqali ishga tushirish uchun konfiguratsiyalar allaqachon sozlangan — breakpoint qo'yish, pauza qilish, o'zgaruvchilarni ko'rish kabi imkoniyatlardan foydalanishingiz mumkin.

Testlarni VS Code'ning Python Tests bo'limi orqali ishga tushirish uchun ham sozlamalar tayyor.

## Docker Compose Override

Development jarayonida `compose.override.yml` faylida faqat lokal development muhitiga ta'sir qiluvchi Docker Compose sozlamalarini o'zgartirishingiz mumkin.

Bu fayldagi o'zgarishlar faqat lokal development muhitiga ta'sir qiladi, production muhitiga emas. Shuning uchun bu yerga development jarayonini osonlashtiruvchi "vaqtinchalik" o'zgarishlarni qo'shishingiz mumkin.

Masalan, backend kodi bo'lgan katalog Docker konteynerida sinxronlashtiriladi — siz o'zgartirgan kod konteyner ichidagi katalogga jonli nusxalanadi. Bu Docker image'ni qayta build qilmasdan o'zgarishlaringizni darhol sinab ko'rish imkonini beradi. Bu faqat development uchun qilinishi kerak; production uchun backend kodining yangi versiyasi bilan Docker image build qilishingiz kerak. Lekin development paytida bu juda tez iteratsiya qilish imkonini beradi.

Shuningdek, standart `fastapi run` o'rniga `fastapi run --reload` ishga tushiradigan buyruq override'i ham bor. U bitta server jarayonini ishga tushiradi (production'dagidek bir nechta emas) va kod o'zgarganda jarayonni qayta yuklaydi. E'tibor bering: agar sintaksis xatosi bo'lsa va Python faylini saqlasangiz, jarayon buziladi va chiqib ketadi, konteyner to'xtaydi. Shundan so'ng xatoni tuzatib, qaytadan ishga tushirishingiz mumkin:

```console
$ docker compose watch
```

Yana, izohga olingan (commented out) `command` override'i ham bor — uni izohdan chiqarib, standart bo'lganini izohga olishingiz mumkin. Bu backend konteynerini "hech narsa qilmaydigan" jarayon bilan ishga tushiradi, lekin konteynerni tirik saqlaydi. Bu ishlab turgan konteyner ichiga kirib, buyruqlarni bajarish imkonini beradi — masalan, o'rnatilgan bog'liqliklarni sinash uchun Python interpretatorini ishga tushirish yoki o'zgarishlarni aniqlaganda qayta yuklanadigan development serverini ishga tushirish.

Konteyner ichiga `bash` sessiyasi bilan kirish uchun stackni ishga tushiring:

```console
$ docker compose watch
```

va boshqa terminalda, ishlab turgan konteyner ichiga `exec` qiling:

```console
$ docker compose exec backend bash
```

Quyidagiga o'xshash natija ko'rishingiz kerak:

```console
root@7f2607af31c3:/app#
```

Bu sizning konteyneringiz ichida, `root` foydalanuvchi sifatida, `/app` katalogi ostida `bash` sessiyasida ekaningizni bildiradi. Bu katalog ichida yana "app" nomli katalog bor — sizning kodingiz konteyner ichida shu yerda joylashgan: `/app/app`.

Shu yerda debug live-reloading serverini ishga tushirish uchun `fastapi run --reload` buyrug'idan foydalanishingiz mumkin.

```console
$ fastapi run --reload app/main.py
```

...natija quyidagicha ko'rinadi:

```console
root@7f2607af31c3:/app# fastapi run --reload app/main.py
```

va keyin Enter bosing. Bu kod o'zgarishlarini aniqlaganda avtomatik qayta yuklanadigan live-reloading serverni ishga tushiradi.

Shunga qaramay, agar u o'zgarishni emas, balki sintaksis xatosini aniqlasa, xato bilan to'xtaydi. Lekin konteyner hali ham tirik va siz Bash sessiyasidasiz, shuning uchun xatoni tuzatgach, bir xil buyruqni ("yuqoriga strelka" va Enter) ishga tushirib, tezda qayta boshlashingiz mumkin.

...aynan shu tafsilot konteynerni hech narsa qilmasdan tirik ushlab turish, so'ngra Bash sessiyasida live-reload serverini ishga tushirishning foydaliligini ko'rsatadi.

## Backend testlari

Backendni sinash uchun quyidagini ishga tushiring:

```console
$ bash ./scripts/test.sh
```

Testlar Pytest yordamida ishlaydi, testlarni `./backend/tests/` katalogida o'zgartiring yoki qo'shing.

Agar GitHub Actions ishlatilsa, testlar avtomatik ishga tushadi.

### Ishlab turgan stack bilan testlash

Agar stackingiz allaqachon ishga tushgan bo'lsa va faqat testlarni ishga tushirmoqchi bo'lsangiz, quyidagidan foydalaning:

```bash
docker compose exec backend bash scripts/tests-start.sh
```

`/app/scripts/tests-start.sh` skripti qolgan stack ishlab turganiga ishonch hosil qilgach, oddiygina `pytest`ni chaqiradi. Agar `pytest`ga qo'shimcha argumentlar uzatish kerak bo'lsa, ularni shu buyruqqa uzatishingiz mumkin — ular pytest'ga yo'naltiriladi.

Masalan, birinchi xatoda to'xtash uchun:

```bash
docker compose exec backend bash scripts/tests-start.sh -x
```

### Test qamrovi (Coverage)

Testlar ishga tushirilganda `htmlcov/index.html` fayli generatsiya qilinadi — uni brauzeringizda ochib, testlar qamrovini ko'rishingiz mumkin.

## Migratsiyalar

Lokal development paytida app katalogingiz konteyner ichiga volume sifatida ulanganligi sababli, migratsiyalarni ham `alembic` buyruqlari orqali konteyner ichida ishga tushirishingiz mumkin va migratsiya kodi (faqat konteyner ichida emas) sizning app katalogingizda ham bo'ladi. Shunday qilib uni git repositoriyangizga qo'shishingiz mumkin.

Modellaringizning "revision"ini yaratganingizga va modellarni har o'zgartirganingizda ma'lumotlar bazangizni shu revision bilan "upgrade" qilganingizga ishonch hosil qiling. Chunki aynan shu narsa ma'lumotlar bazangizdagi jadvallarni yangilaydi. Aks holda ilovangizda xatolar bo'ladi.

* Backend konteynerida interaktiv sessiya boshlang:

```console
$ docker compose exec backend bash
```

* Alembic allaqachon SQLModel modellaringizni `./backend/app/models.py` dan import qilishga sozlangan.

* Modelni o'zgartirgandan so'ng (masalan, ustun qo'shgandan keyin), konteyner ichida revision yarating, masalan:

```console
$ alembic revision --autogenerate -m "Add column last_name to User model"
```

* Alembic katalogida generatsiya qilingan fayllarni git repositoriyaga commit qiling.

* Revisionni yaratgandan so'ng, ma'lumotlar bazasida migratsiyani ishga tushiring (aynan shu narsa ma'lumotlar bazasini haqiqatan ham o'zgartiradi):

```console
$ alembic upgrade head
```

Agar migratsiyalardan umuman foydalanmoqchi bo'lmasangiz, `./backend/app/core/db.py` faylidagi quyidagi qator bilan tugaydigan izohlangan qatorlarni izohdan chiqaring:

```python
SQLModel.metadata.create_all(engine)
```

va `scripts/prestart.sh` faylidagi quyidagi qatorni o'z ichiga olgan qatorni izohga oling:

```console
$ alembic upgrade head
```

Agar standart modellardan boshlashni xohlamasangiz va ularni boshidanoq olib tashlamoqchi/o'zgartirmoqchi bo'lsangiz, hech qanday oldingi revisionsiz, `./backend/app/alembic/versions/` katalogi ostidagi revision fayllarini (`.py` Python fayllarini) o'chirib tashlashingiz mumkin. Va keyin yuqorida tasvirlangandek birinchi migratsiyani yarating.

## Email Shablonlari

Email shablonlari `./backend/app/email-templates/` katalogida joylashgan. Bu yerda ikkita katalog bor: `build` va `src`. `src` katalogi yakuniy email shablonlarini yaratish uchun ishlatiladigan manba fayllarni o'z ichiga oladi. `build` katalogi ilova tomonidan ishlatiladigan yakuniy email shablonlarini o'z ichiga oladi.

Davom etishdan oldin, VS Code'da [MJML kengaytmasi](https://github.com/mjmlio/vscode-mjml) o'rnatilganiga ishonch hosil qiling.

MJML kengaytmasini o'rnatgandan so'ng, `src` katalogida yangi email shabloni yaratishingiz mumkin. Yangi email shablonini yaratib, `.mjml` faylini muharringizda ochgandan so'ng, `Ctrl+Shift+P` bilan buyruqlar palitrasini oching va `MJML: Export to HTML` ni qidiring. Bu `.mjml` faylini `.html`ga aylantiradi va endi uni `build` katalogiga saqlashingiz mumkin.
