# 영화 리뷰 극성 분류 → 감정 분류

NLP 미니프로젝트#1. 네이버 영화 리뷰를 긍정/부정 2개로만 나누던 KoELECTRA 코드(Before)를, 한 리뷰에 여러 감정이 붙을 수 있는 8개 감정 범주 분류 코드(After)로 바꾼다.

용어는 [GLOSSARY.md](GLOSSARY.md)를 따른다.

## Before → After

| 바뀐 지점 | Before (극성 분류) | After (감정 분류) |
|---|---|---|
| ① 출력 수 | 2 (부정, 긍정) | 8 (기쁨·슬픔·분노·공포·놀람·혐오·지루함·중립) |
| ② 활성화·손실 | softmax + cross-entropy (하나만 고름) | sigmoid + BCE (범주마다 따로 판단, 다중 라벨) |
| ③ 정답 형태 | 정수 0/1 | 8칸 0/1 벡터 |
| ④ 학습 순서 | 사전학습 → NSMC | 사전학습 → NSMC → KOTE (**2단계 학습**) |

비교 실험: NSMC를 거치지 않고 사전학습 모델에서 바로 KOTE를 학습한 **직접 학습** 모델과 macro F1을 비교한다 (시드 3개, 평균 ± 표준편차).

## 노트북

| 노트북 | 내용 | 결과 | Colab |
|---|---|---|---|
| [01_nsmc_polarity](notebooks/01_nsmc_polarity.ipynb) | Before: KoELECTRA로 NSMC 극성 분류 재현 | test 정확도 **91.09%** (보고치 90.63%) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tlgus2882-ctrl/nlp_miniproject/blob/main/notebooks/01_nsmc_polarity.ipynb) |
| [02_kote_eda_mapping](notebooks/02_kote_eda_mapping.ipynb) | KOTE 분석, 정답 기준 결정, 44개 원본 라벨 → 8개 감정 범주 변환 | train/val/test 38,687 / 4,835 / 4,844건 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tlgus2882-ctrl/nlp_miniproject/blob/main/notebooks/02_kote_eda_mapping.ipynb) |
| [03_direct_vs_two_stage](notebooks/03_direct_vs_two_stage.ipynb) | After: 직접 학습 vs 2단계 학습 (2방식 × 시드 3개, max_len 128) | 실행 후 기록 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tlgus2882-ctrl/nlp_miniproject/blob/main/notebooks/03_direct_vs_two_stage.ipynb) |
| [04_final_model](notebooks/04_final_model.ipynb) | 최종 모델: 2단계 학습 시드 선택, 범주별 threshold 튜닝, NSMC 데모·극성 일관성 점검 | 실행 후 기록 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tlgus2882-ctrl/nlp_miniproject/blob/main/notebooks/04_final_model.ipynb) |

01·03은 GPU(Colab T4)가 필요하고, 02는 CPU로 돈다. 03은 01이 Drive에 저장한 NSMC 모델을, 04는 03이 Drive에 남긴 결과를 불러온다. 04의 1~2절은 CPU로 돈다.

## 데이터

| | NSMC | KOTE |
|---|---|---|
| 용도 | 극성 분류 | 감정 분류 |
| 출처 | 네이버 영화 리뷰 | 12개 플랫폼 온라인 댓글 (영화 리뷰 포함) |
| 규모 | train 15만 / test 5만 | train 4만 / val 5천 / test 5천 |
| 라벨 | 긍정·부정 | 44개 감정, 평가자 5명 |

KOTE 정답은 제공되는 `dichotomized` 라벨 대신 **평가자 5명 중 2명 이상이 고른 감정**으로 다시 만들었다. `dichotomized`는 1명만 고른 감정도 정답으로 넣어서, 그대로 쓰면 댓글 하나에 평균 2.90개 범주가 켜지고 기쁨·슬픔이 절반 넘는 댓글에 동시에 붙는다. 근거와 대응표는 노트북 02에 있다.

## 참고

- 데이터: [e9t/nsmc](https://github.com/e9t/nsmc), [searle-j/KOTE](https://github.com/searle-j/KOTE)
- 모델: [monologg/KoELECTRA](https://github.com/monologg/KoELECTRA) (`koelectra-base-v3-discriminator`), 비교용 [SKTBrain/KoBERT](https://github.com/SKTBrain/KoBERT)
- 논문: [Clark et al. (2020), ELECTRA](https://arxiv.org/abs/2003.10555), [Jeon et al. (2022), User Guide for KOTE](https://arxiv.org/abs/2205.05300)
