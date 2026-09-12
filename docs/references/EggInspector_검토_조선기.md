# EggInspector 참고 프로젝트 검토

- 정리 담당자: 조선기
- 검토일: 2026-09-12
- 원저자 / 저장소: [So1pi/EggInspector](https://github.com/So1pi/EggInspector)
- 기준 커밋: `13d9a8ab5860cd67ab227ad7cb05b5878ce71273` (master)
- 목적: AI 기반 파각 검출 및 검란 기능 개발을 위한 외부 구현 사례 조사
- 상태: 정적 코드 검토 완료. 학습 실행·성능 재현은 미수행.

## 참고 범위

| 항목 | 확인한 구현 | 검토할 내용 |
| --- | --- | --- |
| 모델 | Faster R-CNN, ResNet-50 FPN | 계란 위치·상태 검출의 비교 기준 |
| 데이터 | AI Hub COLOR 이미지와 XML 라벨 | 이미지-XML 연결, 클래스 매핑, 박스 파싱 |
| 구조 | Dataset, DataLoader, Trainer 분리 | 데이터 처리와 학습의 역할 분리 |
| 체크포인트 | epoch별 모델·옵티마이저 저장 | 학습 재개와 실험 재현 |

함수명에 segmentation이 있지만 실제 모델은 Faster R-CNN입니다. 마스크 분할 구현으로 해석하지 않습니다.

## 적용 전 점검

1. **이미지·박스 동시 변환**: EggDataset은 이미지에 RandomRotation을 적용하지만 박스 좌표를 함께 갱신하지 않습니다.
2. **검증 변환 분리**: 같은 Dataset을 random_split하여 검증 데이터에도 무작위 증강이 적용됩니다. 고정 변환·분할과 동일 계란·촬영 묶음의 누출 여부를 확인합니다.
3. **저장 경로**: Trainer는 최상위 save_dir를 읽지만 JSON에는 trainer.save_dir에 있습니다. 저장 디렉터리 생성도 확인합니다.
4. **배치 크기**: JSON의 data_loader.batch_size는 1024지만 train.py가 사용하는 최상위 batch_size는 4입니다.
5. **빈 라벨**: Dataset의 `(None, None)` 반환은 collate_fn의 `x is not None` 검사로 제외되지 않습니다. 박스 shape·라벨 범위·target 구조를 검증합니다.
6. **평가**: 실제 라벨 명세로 클래스·배경 매핑을 확인하고 mAP, 클래스별 recall, 파각 누락률, 추론 시간을 자체 시험 데이터에서 측정합니다.

## 출처와 재사용

외부 코드 조사 기록이며 팀 자체 개발 코드로 표시하지 않습니다. 확인한 커밋에는 LICENSE 파일이 없습니다. 코드를 가져올 경우 재사용 조건을 먼저 확인하고 원본 URL·커밋·가져온 파일·수정 내용·작업자를 기록합니다.

## 외부 데이터

- [AI Hub 계란 데이터 71504](https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&dataSetSn=71504)
- 외부데이터이며 기업 제공 원본이 아닙니다.
- 수집 담당자: 조선기
- Drive: `02_데이터/02_raw_외부데이터/데이터 수집_조선기`
- 실제 다운로드 완료 여부는 Drive 수집 기록에서 확인합니다.

## 후속 작업

- [ ] 담당자·검토자 확정
- [ ] 클래스 매핑과 XML 표본 검증
- [ ] 이미지·박스 동시 변환 및 검증 분할 설계
- [ ] 소량 데이터 학습·저장·재개 확인
- [ ] 별도 시험 데이터의 성능·추론 시간 비교
- [ ] 도입 부분과 자체 구현 부분을 보고서에 구분
