# HyperExecute - Huong dan chay va hieu luong thuc thi

Tai lieu nay ghi lai cach chay HyperExecute trong project nay, luong thuc thi tu luc go lenh den khi job `COMPLETED`, va y nghia cac thong so tong ket.

## 1) Chuan bi truoc khi chay

- Dang dung Windows trong thu muc project: `D:\OTHERS\Downloads\Selenium_Java\testng-selenium-hyperexecute-sample`
- Co file `hyperexecute.exe` trong root project
- Da set bien moi truong LambdaTest:

```powershell
$env:LT_USERNAME="your_username"
$env:LT_ACCESS_KEY="your_access_key"
```

> Neu dung CMD:
>
> ```cmd
> set LT_USERNAME=your_username
> set LT_ACCESS_KEY=your_access_key
> ```

## 2) Lenh chay

Chay voi file config root:

```powershell
.\hyperexecute.exe --config hyperexecute.yaml --force-clean-artifacts --download-artifacts
```

Hoac chay truc tiep file sample:

```powershell
.\hyperexecute.exe --config yaml/win/v1/testng_hyperexecute_autosplit_sample.yaml --force-clean-artifacts --download-artifacts
```

## 3) Luong thuc thi thuc te (autosplit)

Voi `hyperexecute.yaml` hien tai, job duoc chay theo thu tu sau:

1. CLI doc `hyperexecute.yaml` + xac thuc `LT_USERNAME`, `LT_ACCESS_KEY`.
2. CLI dong goi source code + yaml (payload) va upload len HyperExecute.
3. HyperExecute tao pipeline theo config:
   - `runson: win`
   - `runtime: java 11`
   - `cacheDirectories: .m2`
4. Chay stage `pre`:
   - `mvn -Dmaven.repo.local=./.m2 dependency:resolve`
5. Chay stage `discovery` (autosplit):
   - Lenh: `grep 'test name' xml/testng_win.xml | awk '{print$2}' | sed 's/name=//g' | sed 's/\x3e//g'`
   - Ket qua discover: `Test_1`, `Test_2`, `Test_3`, `Test_4`.
6. Vi `autosplit: true`, HyperExecute tao task theo tung gia tri discover.
7. Vi `concurrency: 4`, cac task co the chay song song toi da 4 task.
8. Moi task chay `testRunnerCommand` voi bien `$test` khac nhau:
   - Task 1: `-DselectedTests=Test_1`
   - Task 2: `-DselectedTests=Test_2`
   - Task 3: `-DselectedTests=Test_3`
   - Task 4: `-DselectedTests=Test_4`
9. Maven Surefire doc `xml/testng_${platname}.xml` (voi `platname=win` => `xml/testng_win.xml`) va chi chay dung `<test name="...">` tuong ung.
10. HyperExecute tong hop log, artifact, va in summary `COMPLETED`.

## 4) Mapping test name -> class trong project nay

Trong `xml/testng_win.xml`:

- `Test_1` -> class `Test1`
- `Test_2` -> class `Test2`
- `Test_3` -> class `Test3`
- `Test_4` -> class `Test4`

Nghia la autosplit dang tach theo `<test name="...">` trong XML, khong phai tu dong tach theo tat ca class trong source.

## 5) Y nghia cac thong so trong phan COMPLETED

Vi du summary da gap:

- `Test Execution Time: 2m20s`
  - Tong thoi gian chi tinh giai doan test (`"Test_1"... "Test_4"`), khong tinh upload/setup.

- `Job Duration Time: 3m43s`
  - Tong thoi gian toan job tu bat dau den ket thuc (co setup, pre, discovery, test, overhead).

- `Total Tasks: 4`
  - Tong so task tao ra tu autosplit (4 test names).

- `Cumulative Time Consumed: 6m31s`
  - Tong cong don thoi gian task/stage tren ha tang. Co the lon hon `Job Duration Time` vi chay song song.

- `Total Stages: 12`
  - 4 tasks x 3 stage (`pre`, `discovery`, `test`) = 12.

- `Pass discovery/pre/test stage percentage: 100%`
  - Tat ca cac stage tuong ung deu pass.

- `Failed ... percentage: 0%`
  - Khong co stage nao fail.

- `Pass post stage percentage: 0%` va `Failed post stage percentage: 0%`
  - Khong cau hinh `post`, nen khong co stage post duoc chay.

- `Failed Tasks: 0`
  - Khong co task fail, job overall thanh cong.

## 6) Khi gap loi thuong gap

- `ERR::NO::USER Unable to find LT username`
  - Chua set `LT_USERNAME`/`LT_ACCESS_KEY` hoac set sai shell session.

- `ERR::NO::HTY Unable to find hyperexecute config file`
  - Sai duong dan `--config` hoac file khong ton tai tai vi tri do.

- `'.\hyperexecute.exe' is not recognized`
  - Chua co binary trong thu muc hien tai hoac go sai ten file.

## 7) Checklist chay nhanh moi lan

1. Mo terminal tai root project.
2. Set `LT_USERNAME`, `LT_ACCESS_KEY`.
3. Chay:
   - `.\hyperexecute.exe --config hyperexecute.yaml --force-clean-artifacts --download-artifacts`
4. Mo Job Link tren dashboard de xem timeline tung stage/task.
5. Kiem tra phan `COMPLETED` de doi chieu toc do va ket qua.

