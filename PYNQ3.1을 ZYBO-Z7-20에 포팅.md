네, PYNQ v3.1을 ZYBO-Z7-20 보드에 포팅하는 작업은 하드웨어 설계와 소프트웨어 빌드를 모두 포함하는 과정입니다. 확보하신 board file과 XDC 파일 외에 필요한 파일 목록과 단계별 포팅 순서를 자세히 안내해 드리겠습니다.

---

## **📥 추가로 필요한 파일 및 생성해야 할 파일 목록**

가지고 계신 파일 외에, 포팅 과정에서 **직접 생성하거나 PYNQ 소스코드로부터 확보**해야 할 핵심 파일들은 다음과 같습니다.

1. **베이스 오버레이 Vivado 프로젝트 (TCL 스크립트):**  
   * ZYBO-Z7-20의 하드웨어 구성을 정의하는 Vivado 프로젝트가 필요합니다. PYNQ-Z2와 같은 기존 보드의 .tcl 파일을 수정하여 ZYBO-Z7-20에 맞게 만드는 것이 가장 효율적입니다.  
2. **PYNQ GitHub 리포지토리 소스 코드:**  
   * PYNQ 이미지 빌드를 위한 전체 소스 코드와 빌드 시스템(sdbuild)이 필요합니다. git을 통해 특정 버전(v3.1)의 소스를 받아야 합니다.  
3. **하드웨어 사양 파일 (.xsa):**  
   * Vivado에서 ZYBO-Z7-20용 하드웨어 설계를 완료한 후 생성되는 출력 파일입니다. 이 파일은 Petalinux(PYNQ의 OS 빌드 도구)가 하드웨어를 인식하는 데 사용됩니다.  
4. **보드 스펙 파일 (.spec):**  
   * PYNQ 빌드 시스템에게 ZYBO-Z7-20 보드에 대한 빌드 방법을 알려주는 설정 파일입니다. PYNQ 소스 내 boards 폴더에 직접 생성해야 합니다.

---

## **⚙️ PYNQ v3.1 포팅 단계별 순서**

포팅 과정은 크게 **\[환경 준비\] → \[하드웨어 설계\] → \[PYNQ 이미지 빌드\] → \[배포 및 테스트\]** 4단계로 나뉩니다.

### **1단계: 개발 환경 준비**

가장 중요한 단계는 **버전 호환성**을 맞추는 것입니다. PYNQ v3.1은 특정 버전의 Xilinx 툴과 동기화되어 있습니다.

1. **Xilinx 툴 설치:**  
   * **Vivado / Vitis / Petalinux 2023.1 버전**을 설치하는 것을 권장합니다. PYNQ v3.1은 2023.1 버전과 공식적으로 호환됩니다. 버전이 다를 경우 빌드 과정에서 예측 불가능한 오류가 발생할 수 있습니다.  
2. **PYNQ 소스 코드 다운로드:**  
   * Git을 사용하여 PYNQ v3.1 태그 버전의 소스 코드를 복제(clone)합니다.

Bash  
git clone https://github.com/Xilinx/PYNQ.git  
cd PYNQ  
git checkout v3.1.0 \-b v3.1.0

3. **ZYBO-Z7-20 보드 파일 설치:**  
   * 확보하신 보드 파일을 Vivado 설치 경로 내의 data/boards/board\_files 폴더에 복사하여 Vivado가 보드를 인식할 수 있도록 합니다.

### **2단계: Vivado를 이용한 하드웨어 설계**

PYNQ의 기본 하드웨어(베이스 오버레이)를 ZYBO-Z7-20에 맞게 수정하는 과정입니다.

1. **참조 프로젝트 Tcl 스크립트 수정:**  
   * PYNQ 소스 코드의 boards/Pynq-Z2/base 폴더에서 base.tcl 파일을 복사합니다.  
   * 이 파일을 열어 프로젝트 이름과 보드 파트 이름을 pynq-z2에서 zybo-z7-20으로 변경합니다.

Tcl  
\# 예시: base.tcl 파일 수정  
\# create\_project \<프로젝트\_이름\> ./\<프로젝트\_이름\> \-part \<FPGA\_파트\>  
\# set\_property board\_part tul.com.tw:pynq-z2:part0:1.0 \[current\_project\]  
\# 위 라인을 아래와 같이 수정  
set\_property board\_part digilentinc.com:zybo-z7\-20:part0:1.2 \[current\_project\]

2. **Vivado 프로젝트 생성:**  
   * Vivado Tcl 콘솔에서 수정된 base.tcl 스크립트를 실행하여 ZYBO-Z7-20 기반의 프로젝트를 자동으로 생성합니다.

Tcl  
source \<수정된\_base.tcl\_파일\_경로\>

3. **Zynq 프로세서 설정 적용:**  
   * 생성된 프로젝트에서 Block Design을 엽니다.  
   * **ZYNQ7 Processing System** 블록을 더블 클릭한 후, 상단의 초록색 배너에서 **Run Block Automation**을 실행합니다. 이때 ZYBO-Z7-20 보드의 프리셋(Preset)이 적용되어 DDR 메모리 및 MIO 핀 설정이 자동으로 구성됩니다.  
4. **외부 포트 및 제약 조건(XDC) 연결:**  
   * Block Design의 외부 포트(예: leds\_4bits\_tri\_o, btns\_4bits\_tri\_i 등)가 ZYBO-Z7-20의 실제 핀에 맞게 연결되도록 **확보하신 XDC 파일을 프로젝트에 추가**하고 설정합니다. PYNQ-Z2와 ZYBO-Z7-20의 스위치 개수(2개 vs 4개) 등 차이가 있으므로, Block Design에서 포트 수를 맞게 수정하고 XDC 파일과 일치시켜야 합니다.  
5. **하드웨어 사양 파일(.xsa) 생성:**  
   * 디자인 검증(Validate Design) 후, **Generate Bitstream**을 실행하여 합성과 구현을 완료합니다.  
   * 성공적으로 완료되면 File \-\> Export \-\> Export Hardware 메뉴를 통해 **Include bitstream** 옵션을 선택하고 .xsa 파일을 생성합니다. 이 파일은 3단계에서 사용될 핵심 결과물입니다.

### **3단계: PYNQ 빌드 시스템을 이용한 이미지 빌드 🚀**

PYNQ의 sdbuild 시스템을 사용하여 ZYBO-Z7-20 전용 부팅 SD카드 이미지를 만듭니다.

1. **ZYBO-Z7-20 보드 폴더 생성:**  
   * PYNQ 소스 코드의 boards 폴더 안에 Zybo-Z7-20이라는 새 폴더를 만듭니다.  
2. **필수 파일 배치:**  
   * boards/Zybo-Z7-20 폴더 안에 다음 파일들을 준비합니다.  
     * **Zybo-Z7-20.spec:** boards/Pynq-Z2/pynq-z2.spec 파일을 복사하여 만듭니다. 파일 내용은 ZYBO-Z7-20에 맞게 수정합니다 (예: BSP\_Zybo-Z7-20 등). 이 파일은 빌드 설정을 담고 있습니다.  
     * **petalinux\_bsp.tcl:** Pynq-Z2 폴더에서 복사해 온 후, 내부의 .xsa 파일 경로 등을 수정합니다.  
     * **\<이름\>.xsa:** 2단계에서 생성한 하드웨어 사양 파일을 이 폴더로 복사합니다.  
     * **base 폴더:** 베이스 오버레이 관련 파일(bitstream, tcl, hwh)을 담을 폴더입니다. Pynq-Z2의 구조를 참고하여 구성합니다.  
3. **PYNQ 이미지 빌드 실행:**  
   * PYNQ 소스 코드의 sdbuild 폴더로 이동하여 make 명령어를 실행합니다.

   Bash  
     cd \<PYNQ\_소스\_경로\>/sdbuild  
     \# 빌드 전 Petalinux 툴 환경 설정  
     source \<Petalinux\_설치\_경로\>/settings.sh  
     \# ZYBO-Z7-20 보드만 타겟으로 하여 빌드 시작  
     make BOARDS=Zybo-Z7-20

   * **주의:** 빌드 과정은 시스템 사양에 따라 **수 시간 이상** 소요될 수 있으며, 수십 GB의 디스크 공간이 필요합니다.

### **4단계: 배포 및 테스트**

1. **SD카드 이미지 플래싱:**  
   * 빌드가 성공적으로 완료되면 sdbuild/output 폴더에 Zybo-Z7-20-v3.1.0.img와 같은 이미지 파일이 생성됩니다.  
   * BalenaEtcher나 Raspberry Pi Imager와 같은 툴을 사용하여 이 .img 파일을 SD카드에 플래싱합니다.  
2. **부팅 및 테스트:**  
   * 이미지가 구워진 SD카드를 ZYBO-Z7-20에 삽입하고 부팅합니다.  
   * 이더넷을 연결한 후, 웹 브라우저에서 http://pynq:9090 또는 http://\<보드\_IP\_주소\>:9090으로 접속하여 Jupyter Lab 환경에 접근합니다.  
   * 간단한 Python 코드로 LED나 버튼을 제어하는 예제를 실행하여 베이스 오버레이와 하드웨어가 올바르게 포팅되었는지 최종 확인합니다.

이 과정을 통해 ZYBO-Z7-20에서 동작하는 PYNQ v3.1 환경을 구축할 수 있습니다. 각 단계에서 발생하는 문제를 해결하며 진행하는 것이 일반적이므로, PYNQ 공식 문서나 커뮤니티 포럼을 함께 참고하시는 것이 좋습니다.