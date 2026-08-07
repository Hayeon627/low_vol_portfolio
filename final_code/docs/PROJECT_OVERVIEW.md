# 성향별 액티브 ETF 펀드 — Low-Risk BL 프로젝트

> **저변동성 anomaly + LSTM σ 예측 + Black-Litterman 단일 view 프레임워크로 위험성향별 펀드 후보를 탐색하는 quant 프로젝트.**
>
> 최종 갱신: 2026-08-07

> ### 📍 계보: 1단계 (K_CUT / hold-out 검증 설계)
> 본 문서는 **1단계** 산출물을 기술한다. 국면 정의는 R1 30m / R2 90m / R3 48m(~2023-12-31) + R4 = hold-out 24m 이며, 상수는 `master_table.REGIMES` 이다.
> **논문의 최종 결론은 2단계(full)** 로, 단일 winner 대신 두 옵션(Ensemble-defensive / Ensemble-adaptive)을 제안하고 국면을 R1 30m / R2 90m / R3 42m / R4 30m 로 재정의한다.
> 1단계 winner `mat_eq_eq_raw_pap` 는 hold-out 24개월에서 Sortino 0.516 (90개 중 88위) 으로 실패했으며, 이 negative result 가 2단계 설계 전환의 근거다.
> 배경은 저장소 루트 [README.md](../../README.md) "분석 계보 — 2단계 설계" 절 참조.

---

## 파이프라인 (7-Step)

```
1. Data Collection      ✅ 01_DataCollection.ipynb              — DATA_COLLECTION.md
2. EDA (시계열)          ✅ 02a_EDA_Returns_Volatility.ipynb     — 수익률 예측 불가 / 변동성 예측 가능 증명
2. EDA (횡단면)          ✅ 02b_LowVol_PortfolioSort.ipynb           — ANOMALY_ANALYSIS.md
3. LSTM HPO             ✅ 03a_LSTM_Optuna_GridSearch.ipynb     — V4_BEST_CONFIG 정당화 (Optuna 12-trial)
3. LSTM σ 예측           ✅ 03b_Volatility_Forecasting.ipynb     — LSTM + HAR + Performance ensemble (617 종목 stockwise)
4. Black-Litterman       ✅ 04_BL_Walkforward.ipynb (walk_forward)
5. MVO + 위험성향 매핑    ✅ 05b_Analyze.ipynb (분석 완료)
6. 3-레짐 안정성 + 민감도  ✅ 05b_Analyze.ipynb (I/J/K/M/N) + 06_Regime_Analysis.ipynb (4-레짐 hold-out)
7. 대시보드              ✅ root_html/ (Streamlit 프로토타입 정적 export)
```

---

## 파일 구조

`final_code/`는 코드 전용 폴더. 노트북이 생성하는 데이터·결과·차트는 최종 스냅샷 정리 과정에서
저장소 루트의 `final_data/`, `final_outputs/`로 옮겨졌다 (재실행 시에는 아래처럼 `final_code/` 하위에
`data/`, `results/`, `outputs/`가 새로 생성됨 — [BL_EXPERIMENT_GUIDE.md](BL_EXPERIMENT_GUIDE.md) §8 참고).

```
final_code/
├── 01_DataCollection.ipynb            ← 데이터 수집 (S&P500 멤버십 + 가격 + 패널 + 보조)
├── 02a_EDA_Returns_Volatility.ipynb   ← 시계열 EDA (수익률 vs 변동성 예측성)
├── 02b_LowVol_PortfolioSort.ipynb     ← 횡단면 EDA (저변동 anomaly 6단 검증)
├── 03a_LSTM_Optuna_GridSearch.ipynb   ← HPO 보조 (12-trial Optuna → V4_BEST_CONFIG)
├── 03b_Volatility_Forecasting.ipynb   ← LSTM 학습 + HAR baseline + Diebold-Pauly ensemble
├── 04_BL_Walkforward.ipynb            ← walk_forward 실행 → results/*.pkl
├── 05a_HMM_Regime.ipynb               ← 3-레짐 HMM 분류
├── 05a_HMM_Regime_full.ipynb          ← 05a 전체 변수셋 버전 (VIF/구조적 단절 부록)
├── 05b_Analyze.ipynb                  ← 분석 (§1 K_CUT → §2-§6 → §7 BL α 분해 → §8 winner 시계열)
├── 06_Regime_Analysis.ipynb           ← 4-레짐 (3-K_CUT + R4 hold-out) winner 검증
├── 99_analyze_ann.ipynb               ← ANN-anchor 벤치마크 비교
├── 99_analyze_ann_full.ipynb          ← 99_analyze_ann 전체 변수셋 버전
├── 99_main_analysis.ipynb             ← 통합 분석 백업본
│
├── appendix/                          ← Utility / Appendix 노트북 (발표 본문 외)
│   ├── 99_explore.ipynb                    ← 실험 탐색 utility (Q&A 백업)
│   ├── 99_slot_effects.ipynb               ← Winner OAT (One-At-a-Time) 슬롯 효과
│   ├── 99_lstm_statistics.ipynb            ← LSTM RMSE 학술 통계
│   ├── draft_a3_r1_concentration.ipynb     ← R1 구간 집중도 draft
│   └── draft_a4_composition_qstats.ipynb   ← 포트폴리오 구성 Q 통계 draft
│
├── timeseries_lib.py                  ← 시계열·통계 함수 (LSTM, HAR-RV, ensemble, 통계 검정)
├── lstm_pipeline.py                   ← LSTM high-level orchestration (V4_BEST_CONFIG, walk_forward)
├── bl_config.py                       ← EXPERIMENTS 정의 (90개 매트릭스)
├── bl_config_ann.py                   ← ANN-anchor 벤치마크용 EXPERIMENTS 정의
├── bl_functions.py                    ← BL 핵심 함수 (Σ/π/P/Q/Ω/BL/TC/Metrics)
├── bl_runner.py                       ← LSTM 로드 + monthly_cache + walk_forward 엔진
├── master_table.py                    ← results/*.pkl → mt/rt 빌더
├── analyze_plots.py                   ← 시각화 모듈
└── docs/                              ← 이 문서들이 위치한 폴더
```

최종 산출물(데이터·pkl·차트)은 저장소 루트 기준:

```
final_data/data/                       ← 01 산출물(monthly_panel, daily_returns, macro, FF 등) + data/03b_lstm/ (LSTM 예측치, BL pkl 등)
final_outputs/outputs/                 ← 단계별 폴더 (01_data … 06_Regime_Analysis, 99_*) 차트 PNG
root_html/root_files/                  ← 대시보드 정적 export (Adaptive VolControl Fund.html)
```

---

## 문서 가이드

| 목적 | 참고 |
|---|---|
| 실험 슬롯 수식 + 추가법 + 실행법 | [BL_EXPERIMENT_GUIDE.md](BL_EXPERIMENT_GUIDE.md) |
| 데이터 수집 파이프라인 | [DATA_COLLECTION.md](DATA_COLLECTION.md) |
| 저변동 anomaly 검증 EDA | [ANOMALY_ANALYSIS.md](ANOMALY_ANALYSIS.md) |
| Q·PCT 민감도 분석 (winner 슬롯) | [SENSITIVITY_ANALYSIS.md](SENSITIVITY_ANALYSIS.md) |
| 학술 reference (Pyo & Lee 2018) | [Exploiting_LowRisk_Anomaly_BL_Summary.md](Exploiting_LowRisk_Anomaly_BL_Summary.md) |
| Winner BL 슬롯 시계열 추이 (§8) | [WINNER_SLOT_TIMESERIES.md](WINNER_SLOT_TIMESERIES.md) |
| 분석 노트북 셀 해설 | `05b_Analyze.ipynb` 안 markdown 셀 |

---

## 핵심 결과

### 02 EDA 정당화
- **02a**: 수익률 R²≈0 / 변동성 ACF lag 60까지 유의 → **LSTM 변동성 예측 모델 정당화** (→ 03b)
- **02b** (forward portfolio sort, Frazzini-Pedersen 2014 형식): 저변동 Sharpe 0.96 vs 고변동 0.73, MDD -16.7% vs -34.1% → **risk-adjusted 우위 일관 + BL spread view 정당화**

### 03 모델
- **03a** Optuna 12-trial: best `3ch_vix / IS=1250 / embargo=63` → avg RMSE 0.3107 (HAR-RV 0.3477 대비 −10.6% 개선)
- **03b** 617 종목 stockwise walk-forward (225 fold): LSTM 0.5185 / HAR 0.3914 / **Ensemble 0.3822** (Best ensemble 403 종목, 65.3%)

### BL 백테스트
- **Winner** (자동 식별: sortino_ir ≥ 10 필터 + 전체기간 sortino 1위): `mat_eq_eq_raw_pap` (Sharpe 1.096, Sortino 1.826, CAGR 16.2%, MDD -13.6%)
- **baseline 대비** (`mat_mcap_mcap_fix_he`): Sharpe·Sortino·CAGR 모두 우위 + MDD 양호
- **SPY 대비**: 모든 위험조정 지표에서 우위 (특히 위기 방어)
- **Q 민감도**: q ∈ [0.001, 0.010] 전 구간에서 winner 와 통계적 동등 (Memmel JK z-test, p>0.5). BAB 학술값 0.0055/0.0064 도 동등 → 학술 정합
- **레짐 안정성**: 3-레짐 (R1 회복 / R2 확장 / R3 변동) HMM 기반

---

## 명명 체계 — canonical 5-token

```
{prior}_{p_mode}_{p_weight}_{q}_{omega}
mcap   _ls    _eq      _lam_pap  ← 예
```

토큰 정의 + 슬롯 수식 → [BL_EXPERIMENT_GUIDE.md](BL_EXPERIMENT_GUIDE.md) §4 참고
