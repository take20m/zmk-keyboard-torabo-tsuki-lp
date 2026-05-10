# PAW3222 電池起動時トラックボール初期化失敗の自前修正手順

## 症状

- 単3電池のみで起動 → キーは動くがトラックボールが反応しない
- USB給電で起動 → 正常
- 電池起動 → USB挿す → USB抜く → 電池でもトラックボール動く
- 原因: 昇圧コンバータの立ち上がりとPAW3222初期化のタイミング問題

## 関連情報

- Issue: https://github.com/sekigon-gonnoc/torabo-tsuki-lp/issues/2
- ドライバ: https://github.com/sekigon-gonnoc/zmk-driver-paw3222
- 修正対象: `src/paw3222.c`

## 手順

### 1. フォーク

https://github.com/sekigon-gonnoc/zmk-driver-paw3222 をフォーク

### 2. クローン & 修正

```bash
git clone git@github.com:MisonoTakezo/zmk-driver-paw3222.git
cd zmk-driver-paw3222
```

`src/paw3222.c` を編集:

#### 修正A: 電源安定化待ちを増やす（paw32xx_init 内）

```c
// 変更前（約388行目付近）
// Wait 0.01 seconds before turning on power
k_sleep(K_MSEC(10));

// 変更後
// Wait 0.5 seconds for boost converter stabilization
k_sleep(K_MSEC(500));
```

#### 修正B: リトライ回数・間隔を増やす（paw32xx_configure 内）

```c
// 変更前（約322行目付近）
int retry_count = 10;
// ...
k_sleep(K_MSEC(100));

// 変更後
int retry_count = 30;
// ...
k_sleep(K_MSEC(200));
```

### 3. コミット & プッシュ

```bash
git add -A
git commit -m "電池起動時の初期化安定化: 電源待ち増加 + リトライ強化"
git push
```

### 4. west.yml を変更

`config/west.yml` の `zmk-driver-paw3222` のリモートを自分のフォークに向ける:

```yaml
# 変更前
- name: zmk-driver-paw3222
  remote: sekigon-gonnoc
  revision: main

# 変更後
- name: zmk-driver-paw3222
  url: https://github.com/MisonoTakezo/zmk-driver-paw3222
  revision: main
```

### 5. ビルド & 書き込み

```bash
git add config/west.yml
git commit -m "zmk-driver-paw3222を自前フォークに切り替え"
git push
```

GitHub Actions でビルド → firmware をダウンロード → 書き込み

### 6. 動作確認

- 電池のみで起動してトラックボールが動くか確認
- ダメなら `k_sleep` の値をさらに増やす or 案3（遅延再初期化）を検討

## 元に戻す方法

`config/west.yml` を元に戻すだけ:

```yaml
- name: zmk-driver-paw3222
  remote: sekigon-gonnoc
  revision: main
```
