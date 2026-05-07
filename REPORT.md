# Bao Cao Lab Day 21 - CI/CD cho AI Systems

## 1. Thong Tin Chung

- Ho ten: Trịnh Đắc Phú
- Ma hoc vien: 2A202600322
- Project: MLOps pipeline cho bai toan Wine Quality Classification
- Cloud provider: Google Cloud Platform
- Object storage: Google Cloud Storage
- VM serving: Google Compute Engine
- API framework: FastAPI
- CI/CD: GitHub Actions
- Data versioning: DVC
- Experiment tracking: MLflow

## 2. Muc Tieu Lab

Lab nay xay dung mot quy trinh MLOps co kha nang:

1. Theo doi thi nghiem huan luyen model bang MLflow.
2. Quan ly phien ban du lieu bang DVC va Google Cloud Storage.
3. Tu dong chay unit test, train, eval va deploy bang GitHub Actions.
4. Trien khai model len VM duoi dang REST API bang FastAPI.
5. Mo phong continuous training khi co du lieu moi.

## 3. Buoc 1 - MLflow Local Experiment

O buoc nay, toi da hoan thien `src/train.py` de:

- Doc du lieu tu `data/train_phase1.csv` va `data/eval.csv`.
- Huan luyen model.
- Tinh `accuracy` va `f1_score`.
- Log params va metrics vao MLflow.
- Luu metrics vao `outputs/metrics.json`.
- Luu model vao `models/model.pkl`.

Model su dung:

```text
GradientBoostingClassifier
```

Bo sieu tham so trong `params.yaml`:

```yaml
n_estimators: 500
learning_rate: 0.05
max_depth: 6
min_samples_split: 2
```

Ket qua thi nghiem duoc theo doi trong MLflow UI. MLflow UI hien thi nhieu lan chay voi cac bo tham so khac nhau, gom cac chi so `accuracy` va `f1_score`.

Bang chung:

- Screenshot: `screenshots/result_step1.png`

## 4. Buoc 2 - CI/CD, DVC va Deployment

O buoc 2, toi da cau hinh DVC de quan ly cac file du lieu:

```text
data/train_phase1.csv
data/eval.csv
data/train_phase2.csv
```

Du lieu that duoc day len Google Cloud Storage, con Git chi commit cac file pointer `.dvc`:

```text
data/train_phase1.csv.dvc
data/eval.csv.dvc
data/train_phase2.csv.dvc
```

GCS bucket:

```text
mlops-lab-zhengdefu-20260507
```

Workflow GitHub Actions gom 4 job:

```text
Unit Test -> Train -> Eval -> Deploy
```

Trong do:

- `Unit Test`: chay `pytest tests/ -v`.
- `Train`: pull du lieu bang DVC va train model.
- `Eval`: kiem tra nguong chat luong model.
- `Deploy`: SSH vao VM va restart service `mlops-serve`.

Model sau khi train duoc upload len GCS tai:

```text
gs://mlops-lab-zhengdefu-20260507/models/latest/model.pkl
```

API duoc deploy tren VM:

```text
http://34.63.88.37:8000
```

Endpoint kiem tra health:

```bash
curl http://34.63.88.37:8000/health
```

Endpoint predict:

```bash
curl -X POST http://34.63.88.37:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [7.4, 0.70, 0.00, 1.9, 0.076, 11.0, 34.0, 0.9978, 3.51, 0.56, 9.4, 0]}'
```

Bang chung:

- Screenshot GitHub Actions pass: `screenshots/result_step2_step3.png`
- Screenshot API/GCS/git verification: `screenshots/step2_step3_verification_api_gcs_commit.png`

## 5. Buoc 3 - Continuous Training Khi Co Du Lieu Moi

O buoc 3, toi mo phong viec co them du lieu moi bang script:

```bash
python3 add_new_data.py
```

Script nay ghep `data/train_phase2.csv` vao `data/train_phase1.csv`.

Kich thuoc du lieu thay doi:

```text
2998 mau -> 5996 mau
```

Sau do toi cap nhat DVC pointer:

```bash
dvc add data/train_phase1.csv
dvc push
```

Commit du lieu moi:

```text
data: add phase2 training samples
```

Commit nay cap nhat:

```text
data/train_phase1.csv.dvc
```

Khi push commit len GitHub, GitHub Actions tu dong chay lai pipeline:

```text
Unit Test -> Train -> Eval -> Deploy
```

Dieu nay chung minh pipeline co kha nang continuous training: khi du lieu thay doi, he thong tu dong huan luyen lai, kiem tra chat luong va deploy model moi.

Bang chung:

- Screenshot GitHub Actions cho commit du lieu moi: `screenshots/result_step2_step3.png`
- Screenshot `git log --name-only -1`: `screenshots/step2_step3_verification_api_gcs_commit.png`

## 6. Ket Qua Va Nhan Xet

He thong hoan thanh cac yeu cau chinh:

- MLflow ghi lai cac thi nghiem huan luyen model.
- DVC quan ly phien ban du lieu va dong bo len GCS.
- GitHub Actions tu dong chay pipeline CI/CD.
- FastAPI server tren VM phuc vu endpoint `/health` va `/predict`.
- Khi co du lieu moi, commit `.dvc` kich hoat pipeline huan luyen va deploy lai.

Metrics nen duoc lay tu artifact `outputs/metrics.json` trong GitHub Actions cua tung lan chay:

| Lan chay | So mau train | accuracy | f1_score |
|---|---:|---:|---:|
| Buoc 2 | 2998 | Dien tu GitHub Actions artifact | Dien tu GitHub Actions artifact |
| Buoc 3 | 5996 | Dien tu GitHub Actions artifact | Dien tu GitHub Actions artifact |

## 7. Link Repo

GitHub repository:

```text
Dien link GitHub repo cua ban tai day
```
