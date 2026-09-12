# دليل أوامر Git الشامل

مرجع تفصيلي لأهم أوامر git، كل أمر فيه: الفكرة، الصيغة، امتى تستخدمه، وأمثلة.

---

## 1) git help

### الفكرة

الأمر اللي بيوريك توثيق git نفسه.
مفيد لما تنسى صيغة أمر معين.

### الصيغة

```bash
git help
git help <command>
```

مثال:

```bash
git help commit
```

بدائل سريعة:

```bash
git commit --help
git commit -h
```

| الأمر | النتيجة |
|---|---|
| `git help commit` | صفحة توثيق كاملة |
| `git commit --help` | نفس الحاجة |
| `git commit -h` | ملخص سريع في الـ terminal |

### امتى تستخدمه

نسيت flag معين لأمر.
عايز تعرف كل الخيارات المتاحة لأمر معين.

---

## 2) git cherry-pick

### الفكرة

تاخد commit واحد من branch معين.
تحطه في branch تاني.
من غير merge للـ branch كله.

### الصيغة

```bash
git cherry-pick <commit-hash>
```

### امتى تستخدمه

Hotfix عاجل لازم ينزل في `main` فورًا.
commit اتعمل في branch غلط.
عايز commit واحد بس من branch هتتمسح.

### الرسم التوضيحي

قبل:

```
main:     A---B---C
                   \
feature:            D---E---F
```

بعد `git cherry-pick E`:

```
main:     A---B---C---E'
```

### مثال

```bash
git checkout main
git cherry-pick abc1234
```

لو فيه تعارض:

```bash
git add .
git cherry-pick --continue
```

---

## 3) git squash (تقنية مش أمر مستقل)

### الفكرة

دمج كذا commit في commit واحدة.
بتتعمل عن طريق `git rebase -i` أو `git merge --squash`.

### امتى تستخدمه

قبل merge لـ feature branch فيها commits كتير زي "fix typo".
عايز تاريخ نضيف قبل Pull Request.

### مثال: interactive rebase

```bash
git rebase -i HEAD~4
```

يتحول:

```
pick a1b2c3d add login form
pick c3b4a5d add login validation
pick d8e9f1a wip
pick f4a3b2c fix typo
```

لـ:

```
pick a1b2c3d add login form
squash c3b4a5d add login validation
squash d8e9f1a wip
squash f4a3b2c fix typo
```

النتيجة: commit واحدة بدل أربعة.

### الرسم التوضيحي

قبل:

```
main:     A---B
               \
feature:        C---D---E---F
```

بعد `git merge --squash feature`:

```
main:     A---B---G
```

(G = كل تغييرات C وD وE وF مجمّعة)

---

## 4) git merge

### الفكرة

يدمج branch كامل مع branch تاني.
بيحتفظ بتاريخ الـ commits كله.

### الصيغة

```bash
git checkout main
git merge feature-branch
```

### امتى تستخدمه

خلصت شغل الـ feature، عايز تدمجها في main.
عايز تحتفظ بتاريخ التطوير كامل، مش تلخصه.

### الرسم التوضيحي

قبل:

```
main:     A---B
               \
feature:        C---D---E
```

بعد merge (fast-forward مش ممكن هنا، فبيعمل merge commit):

```
main:     A---B-------M
               \      /
feature:        C---D---E
```

M هي الـ merge commit، وليها اتنين parent.

### نوع تاني: fast-forward merge

لو main متغيرتش خالص من وقت ما اتعملت feature:

قبل:

```
main:     A---B
                \
feature:         C---D
```

بعد:

```
main:     A---B---C---D
```

مفيش merge commit هنا، main بس اتحرك قدام.

### التعامل مع تعارض

```bash
git merge feature-branch
# لو فيه conflict
git add .
git commit
```

---

## 5) git rebase

### الفكرة

بياخد commits من branch معين.
ويعيد تطبيقها فوق branch تاني.
النتيجة: تاريخ مستقيم (linear) من غير merge commits.

### الصيغة

```bash
git checkout feature-branch
git rebase main
```

### امتى تستخدمه

عايز تاريخ نضيف مستقيم من غير merge commits.
عايز تحدّث الـ feature branch بآخر تغييرات main قبل ما تعمل merge.

### الرسم التوضيحي

قبل:

```
main:     A---B---C
                   \
feature:            D---E
```

بعد `git rebase main` وانت على feature:

```
main:     A---B---C
                   \
feature:            D'---E'
```

D وE اتعملهم إعادة تطبيق فوق C، بـ hash جديد.

### تحذير مهم

**ممنوع تعمل rebase لـ commits اتعملها push ومشتركة مع ناس تانية.**
ده بيغيّر التاريخ ويعمل مشاكل لزمايلك.

### الفرق بين merge وrebase

| | merge | rebase |
|---|---|---|
| التاريخ | بيحافظ على الشكل الأصلي، فيه merge commits | بيبقى مستقيم، من غير merge commits |
| الأمان مع الفريق | آمن دايمًا | خطر لو الـ commits اتعملها push |
| الاستخدام الشائع | دمج feature كاملة في main | تنضيف الـ branch بتاعتك قبل الدمج |

---

## 6) git clean

### الفكرة

يشيل الملفات اللي مش متتبعة (untracked files).
يعني ملفات موجودة في المجلد بس مش داخلة في git.

### الصيغة

```bash
git clean -n
```

`-n` معناها dry-run، بس بيوريك اللي هيتمسح من غير ما يمسح فعلاً.

للمسح الفعلي:

```bash
git clean -f
```

لمسح المجلدات كمان:

```bash
git clean -fd
```

### امتى تستخدمه

عندك ملفات تجريبية أو build files اتعملت بالغلط ومش عايزها.
عايز تنضف المجلد قبل ما تبدأ شغل جديد.

### مثال

```bash
git status
# untracked files: temp.log, build/

git clean -n
# Would remove temp.log
# Would remove build/

git clean -fd
# Removing temp.log
# Removing build/
```

### تحذير

`git clean -f` بيمسح الملفات نهائيًا، من غير trash أو استرجاع.
اعمل `git clean -n` الأول دايمًا للتأكد.

---

## 7) git grep

### الفكرة

يدور جوه ملفات المشروع (اللي داخل git) عن نص معين.
أسرع من `grep` العادي لأنه بيدور بس في الملفات المتتبعة.

### الصيغة

```bash
git grep "search-term"
```

### امتى تستخدمه

عايز تلاقي فين كلمة أو function معينة متستخدمة في المشروع كله.

### مثال

```bash
git grep "function login"
```

النتيجة:

```
src/auth.js:12:function login(user, password) {
src/routes.js:45:  app.post("/login", function login(req, res) {
```

خيارات مفيدة:

```bash
git grep -n "TODO"        # يوري رقم السطر
git grep -c "import React" # يعد عدد المرات في كل ملف
git grep -i "error"        # case-insensitive
```

### دورة على commit قديم

```bash
git grep "search-term" <commit-hash>
```

---

## 8) git blame

### الفكرة

يوريك مين كتب كل سطر في ملف معين، وفي أي commit.

### الصيغة

```bash
git blame <file>
```

### امتى تستخدمه

عايز تعرف مين غيّر سطر معين وإمتى.
عايز تفهم سياق تغيير معين قبل ما تعدله.

### مثال

```bash
git blame src/auth.js
```

النتيجة:

```
a1b2c3d (Ahmed  2024-03-01) function login(user, password) {
d4e5f6g (Sara   2024-05-10)   if (!user) return null;
```

كل سطر معاه: الـ commit، مين كتبه، وإمتى.

### تضييق النطاق

```bash
git blame -L 10,20 src/auth.js
```

يوري بس السطور من 10 لـ 20.

---

## 9) git bisect

### الفكرة

بيدور على الـ commit اللي سبب bug معين.
عن طريق binary search بين commit شغال وcommit فيه المشكلة.

### الصيغة

```bash
git bisect start
git bisect bad          # الحالة الحالية فيها bug
git bisect good v1.0    # النسخة دي كانت شغالة كويس
```

git هيروح لـ commit في النص، تختبره وتقوله:

```bash
git bisect good   # لو المشكلة مش موجودة هنا
git bisect bad    # لو المشكلة موجودة
```

وهكذا لحد ما يلاقي الـ commit بالظبط.

للخروج:

```bash
git bisect reset
```

### امتى تستخدمه

عندك bug ظهر، بس مش عارف امتى بالظبط دخل المشروع.
عندك تاريخ طويل من الـ commits ومش عملي تراجعهم واحد واحد.

### الرسم التوضيحي

```
good                                    bad
 |                                       |
 A---B---C---D---E---F---G---H---I---J---K
             ^
        git bisect بيقفز هنا الأول (النص بالظبط)
```

كل مرة بيقسم المسافة نص، لحد ما يوصل لـ commit واحد بالظبط.

### مثال كامل

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.0

# git بيوقفك على commit في النص
npm test
# لو الاختبار فشل
git bisect bad
# لو نجح
git bisect good

# يكرر لحد ما يقولك:
# "abc1234 is the first bad commit"

git bisect reset
```

---

## 10) git shortlog

### الفكرة

يلخص الـ commits، مجمعة حسب اسم صاحب كل commit.

### الصيغة

```bash
git shortlog
```

### امتى تستخدمه

عايز تعرف مين ساهم إمتى وبقد إيه في المشروع.
عايز ملخص سريع بدل `git log` الطويل.

### مثال

```bash
git shortlog -sn
```

النتيجة:

```
    45  Ahmed
    30  Sara
    12  Mohamed
```

`-s` يعني summary (عدد بس، من غير رسائل الـ commits).
`-n` يعني ترتيب حسب العدد.

لعرض الرسائل كمان:

```bash
git shortlog
```

```
Ahmed (3):
      fix login bug
      add validation
      update readme

Sara (2):
      add tests
      refactor auth
```

---

## 11) git prune

### الفكرة

يمسح الـ objects في git مش متربوطة بأي commit أو branch.
يعني بيانات "يتيمة" (unreachable) فضلت من عمليات زي rebase أو reset.

### الصيغة

```bash
git prune
```

للتجربة قبل المسح:

```bash
git prune -n
```

### امتى تستخدمه

بعد rebase أو reset كتير، حجم الـ `.git` بيكبر من غير داعي.
عايز تنضف المستودع وتقلل حجمه.

### ملاحظة

في الغالب مش بتستخدمه مباشرة.
الأمر:

```bash
git gc
```

بيستدعي `prune` تلقائيًا كجزء من تنظيف شامل للمستودع.

---

## 12) git worktree

### الفكرة

يخليك تفتح أكتر من نسخة شغالة (working directory) لنفس المستودع.
كل نسخة على branch مختلف، في نفس الوقت، من غير ما تعمل clone تاني.

### الصيغة

```bash
git worktree add ../hotfix-dir hotfix-branch
```

### امتى تستخدمه

شغال على feature branch، وفجأة محتاج تعمل hotfix عاجل على main.
مش عايز تعمل `git stash` أو تفقد شغلك الحالي.
بدل ما تبدل branch، بتفتح مجلد تاني بجانب الأصلي.

### الرسم التوضيحي

```
project/              (main branch, working here)
project-hotfix/        (hotfix-branch, working here too)
```

مجلدين، نفس المستودع (`.git`)، بس كل واحد على branch مختلف.

### مثال كامل

```bash
git worktree add ../project-hotfix hotfix-branch
cd ../project-hotfix
# تشتغل على hotfix هنا من غير ما تلمس شغلك في المجلد الأصلي
```

لعرض كل الـ worktrees:

```bash
git worktree list
```

لحذف worktree بعد ما تخلص:

```bash
git worktree remove ../project-hotfix
```

---

## 13) git verify-commit

### الفكرة

يتأكد إن commit معين موقّع (signed) بـ GPG أو SSH key حقيقي.
بيستخدم للتحقق من هوية صاحب الـ commit.

### الصيغة

```bash
git verify-commit <commit-hash>
```

### امتى تستخدمه

في مشاريع مفتوحة المصدر أو حساسة، بيتم توقيع الـ commits.
عايز تتأكد إن commit معين جاي فعلاً من الشخص اللي بيدعي إنه كتبه.

### مثال

```bash
git verify-commit abc1234
```

لو موقّع وصحيح:

```
gpg: Good signature from "Ahmed <ahmed@example.com>"
```

لو مش موقّع:

```
error: no signature found
```

### شرط أساسي

لازم يكون عندك الـ public key بتاع الشخص ده مضاف عندك محليًا، عشان git يقدر يتحقق من التوقيع.

---

## 14) git filter-repo

### الفكرة

أداة خارجية (مش جزء أساسي من git، لازم تتثبت لوحدها) لإعادة كتابة تاريخ المستودع بالكامل.
بتستخدم لحذف ملفات حساسة أو تعديل تاريخ ضخم بأمان وسرعة.

### التثبيت

```bash
pip install git-filter-repo
```

### امتى تستخدمه

اكتشفت إن ملف فيه password أو API key اتعمله commit بالغلط، وعايز تشيله من **كل** تاريخ المشروع.
عايز تفصل جزء من مستودع كبير في مستودع مستقل.
عايز تقلل حجم مستودع ضخم بحذف ملفات كبيرة قديمة من التاريخ.

### مثال: حذف ملف من كل التاريخ

```bash
git filter-repo --path secrets.env --invert-paths
```

`--path secrets.env` يحدد الملف.
`--invert-paths` معناها "احذف ده من كل حاجة" بدل ما تحتفظ بيه بس.

### الرسم التوضيحي

قبل:

```
A---B---C---D---E
    ^       ^
  فيه secrets.env  فيه تعديل عليه كمان
```

بعد `git filter-repo`:

```
A---B'---C'---D'---E'
```

كل الـ commits اتعملها rewrite، وملف `secrets.env` مش موجود في أي واحدة منهم، حتى القديمة.

### تحذير مهم

ده بيغيّر hash كل الـ commits في المستودع بالكامل.
لازم تنسق مع الفريق كله قبل ما تعمل push، لأن أي نسخة قديمة عند حد هتبقى متعارضة تمامًا.

بعد التنفيذ:

```bash
git push origin --force --all
```
