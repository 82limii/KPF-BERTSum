# KPF-BERTSum

## 모델 소개

### KPF-BERTSum

KPF-BERTSum은 KPF-BERT Text Summarization의 준말로 BERT의 사전학습 모델을 이용한 텍스트 요약 논문 및 모델인 [PRESUMM모델](https://github.com/nlpyang/PreSumm) 모델을 참조하여 한국어 문장의 요약추출을 구현한 한국어 요약 모델이다.

한국언론진흥재단에서 구축한 방대한 뉴스기사 말뭉치로 학습한 KPF-BERT를 이용하여 특히 뉴스기사 요약에 특화된 모델로 한국어 데이터 셋은 AI-HUB에서 제공하는 문서요약 텍스트를 사용하였다.

- 본 예제에 사용된 kpf-BERT는 [kpfBERT](https://github.com/KPFBERT/kpfbert)에 공개되어 있다.

- 한국어 데이터 셋은 ETRI의 AI-HUB에서 제공하는 [문서요약 텍스트(비플라이소프트)](https://aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100)를 사용하였다.  

![img_1](https://user-images.githubusercontent.com/87846939/221451320-442ed501-6bda-4aec-8ac0-bd159800516b.png)


BERTSum은 BERT 위에 inter-sentence Transformer 2-layers를 얹은 구조를 갖습니다. 이를 fine-tuning하여 BertSumExt 요약모델을 사용하였습니다. 
Pre-trained BERT를 summarization task 수행을 위한 embedding layer로 활용하기 위해서는 여러 sentence를 하나의 인풋으로 넣어주고, 각 sentence에 대한 정보를 출력할 수 있도록 입력을 수정해줘야 합니다.
이를 위해 Input document에서 매 문장의 앞에 [CLS] 토큰을 삽입하고, 매 sentence마다 다른 segment embeddings 토큰을 더해주는 interval segment embeddings을 추가하는 구조를 갖게됩니다.


![img](https://user-images.githubusercontent.com/87846939/221451340-0abe8d7a-00ae-499f-bbb2-14d4ac6b2ef3.png)


기존의 BERT를 summariaztion에 바로 적용하기에 BERT는 MLM으로 훈련되기 때문에 출력 벡터가 토큰단위로 출력되게 됩니다.
이러한 한계를 극복하고자 요약 task에서는 문장 수준의 표현을 다루기 위해 BERT의 입력 데이터 형태를 수정하여 사용합니다.

### Bert를 이용한 문서요약 관련 논문 및 코드

- 논문:  [Text Summarization with Pretrained Encoders](https://arxiv.org/abs/1908.08345) (EMNLP 2019 paper)
- 원코드: https://github.com/nlpyang/PreSumm


---
## 실행하기 위한 환경

1. 라이브러리

    ```
    python = 3.9
    pytorch-lightning==1.2.8
    torch==1.12.1
    transformers==4.10.0
    
    그 외 pytorch 등 머신러닝을 위한 기본적인 패키지(소스 내 import 참조)
    ```
    
2. GPU

    ```
    NVIDIA GeForce RTX 3060 환경에서 작성되었음
    ```
---
## Usage

1. 데이터 Preprocessing

   `data/` 디렉토리에 아래의 화일명으로 데이터셋 화일을 넣어준다.

   - `train_original.json`: 트레이닝 데이터로 train과 val 데이터로 8:2 분리하여 사용한다.  
   - `valid_original.json`: 테스트에 사용할 데이터이다.

   
2. Fine-tuning  

    kpfBERT 모델을 다운받아 아래의 디렉토리에 넣어주고 fine-tuning을 진행한다.

    - `kpfbert-base/` kpfbert 다운받은 화일들이 위치할 곳
    - `checkpoints/best-checkpoint.ckpt` fine-tuning 후 최적의 모델이 저장되는 곳.
    - `lightning_logs/kpfBERT_Summary/version_x/` 하위에 fine-tuning 로그가 저장됨, x는 실행순서 번호
    

3. Predictions  

   Fine-tuning된 최적의 model을 불러와서 테스트 문장에 대한 추출요약을 수행하여 결과를 반환한다.

   - `test_context` 테스트 할 기사전문을 입력한다.  
  
    예시)  
      
      입력문장
   ```
   이재명 더불어민주당 대선후보는 26일 변호사비 대납 의혹과 관련, "내가 정말로 변호사비를 불법으로 받았으면 나를 구속하라"고 반박했다.  
   이 후보는 이날 오후 전남 신안군 응급의료 전용헬기 계류장에서 열린 '국민반상회' 후 기자들과 만나 한 시민단체 대표가 고액 수임료 의혹 증거라며 제시한 녹취록에 대해 "조작됐다는 증거를 갖고 있고 검찰에도 제출했다. 검찰과 수사기관들은 빨리 처리하시라"며 이같이 말했다.  
   앞서 이민구 깨어있는시민연대당 대표는 이 후보가 특정 변호사에게 수임료로 현금과 주식 등 20억원을 줬다는 의혹을 주장하며 녹취록을 제출한 바 있다. 이에 대해 송평수 선대위 부대변인은 "허위사실"이라며 "깨시민당 이 대표에게 제보를 했다는 시민단체 대표 이모 씨가 제3자로부터 기부금을 받아낼 목적으로 허위사실을 녹음한 후, 이 모 변호사에게까지 접근했다. 이러한 비상식적이고 악의적인 행태는 이재명 후보에 대한 정치적 타격을 가할 목적으로 치밀하게 준비한 것"이라고 반박했다.  
   이에 대해 이 후보는 "그것도 조직폭력배 조작에 버금가는 조작사건이라는 게 곧 드러날 것"이라며 "팩트확인을 하고 언급하면 좋겠다. 당사자도 아니고 제3자들이 자기끼리 녹음한 게 가치가 있느냐"고 반문했다.  
   그는 "사실이 아니면 무고하고 음해하는 사람들을 무고 혐의나 공직선거법 위반으로 빨리 처리해서 처벌하시라"며 "선거 국면에서 하루이틀도, 한두번도 아니고 '조폭이 뇌물 줬다'는 (허위사실 유포를) 왜 아직도 처리 안 하고 있느냐"고 검경에 불만을 드러냈다.  
   이어 "허위사실이 드러났으면 당연히 다시는 그런 일이 없게 해야 하는 것 아닌가. 이해가 안 된다"며 "선거관리, 또는 범죄를 단속하는 국가기관들이 이런 식으로 허위사실 유포나 무고 행위를 방치해 정치적 공격 수단으로 쓰게 하면 안 된다"고 했다.  
   이 후보는 또 자신이 구민주-동교동계와 접촉해 복당을 타진했다는 언론보도와 관련해선 "구체적으로 어떤 사람을 범주별로 나눠 무슨 계, 진영으로 말하는 것은 아니다"라며 "시점을 언젠가 정해 벌점이니, 제재니, 제한이니 다 없애고 모두가 합류할 수 있도록 할 생각"이라고 말했다.  
   종전에 언급했던 '대사면' 방침을 재확인한 셈이다. 그는 "민주당에 계셨던 분, 또 민주당에 있지 않았더라도 앞으로 함께할 분들에게 계속 연락을 하고 있다"며 "만나고 전화하고 힘을 합치자고 권유하고 있다"고 했다.  
   그는 " 현재 민주당이 이미 열린민주당과의 통합을 협의하고 있다"며 "거기에 더해서 꼭 민주계라고 말할 필요는 없고 부패사범이나 파렴치범으로 탈당하거나 또는 제명된 사람들이 아니라면, 국가의 미래를 걱정하는 민주개혁 진영의 일원이라면 가리지 말고 과거의 어떤 일이든 그러지 말고 힘을 합치자"고 강조했다.  
   언론보도에 따르면, 이 후보는 최근 구민주계인 정대철 전 고문과 연락을 주고 받으며 천정배, 정동영 전 의원 등 민주당을 탈당했던 옛 동교동계와 호남 인사들의 복당을 타진했다.  
   ```
       
     결과값
   ```
    *** 입력한 문단의 요약문은 ...
    [ 1 ] 이재명 더불어민주당 대선후보는 26일 변호사비 대납 의혹과 관련, "내가 정말로 변호사비를 불법으로 받았으면 나를 구속하라"고 반박했다.  
    [ 2 ] 이 후보는 이날 오후 전남 신안군 응급의료 전용헬기 계류장에서 열린 '국민반상회' 후 기자들과 만나 한 시민단체 대표가 고액 수임료 의혹 증거라며 제시한 녹취록에 대해 "조작됐다는 증거를 갖고 있고 검찰에도 제출했다.  
    [ 3 ] 검찰과 수사기관들은 빨리 처리하시라"며 이같이 말했다.  
   ```
4. Module 사용

   `predict_module.py`의 `summarize_test(text)`를 사용하여 KPF-BERT와 생성된 KPF-BERTSum모델을 등록하여 요약추출을 사용할 수 있다.
