---
title:  "Symphonic MPD [3] 지터, 마스터, 슬레이브, 동기, 비동기 전송..."
excerpt: "네트웍 플레이어나 DAC 와 같은 디지털 오디오 소스에 대한 자료를 보다 보면 지터(Jitter) 라는 말이 참 많이 나옵니다. 동기, 비동기 전송은 IT에서도 많이 사용하는 개념이라서 이해하는데 어려움이 없는데 도대체 지터가 뭐길래..."

categories:
  - 라즈베리파이
tags:
  - [라즈베리파이, MPD]

toc: true
toc_sticky: true
 
date: 2021-10-30
last_modified_at: 2021-10-30
---
# 지터 (Jitter)
네트웍 플레이어나 DAC 와 같은 디지털 오디오 소스에 대한 자료를 보다 보면 지터(Jitter) 라는 말이 참 많이 나옵니다. 동기, 비동기 전송은 IT에서도 많이 사용하는 개념이라서 이해하는데 어려움이 없는데 도대체 지터가 뭐길래 음질에 악영향을 주니...어쩌니 해서...참 궁금했습니다.​

또한 유명 제조사의 제품 소개 등에 보면 지터를 획기적으로 개선해서...어쩌구...블라블라....???

그래서 검색을 하면서 내용을 확인했습니다.

지터란, 완벽히 주기적이어야 할 신호가 각 주기마다 조금씩 차이가 날 때 그 차이(편차)를 지터라 부른다. Julian Dunn 은 지터를 " Deviation in timing of transitions when measured with respect to an ideal clock" 라 하여 주기적인 신호가 주기를 시작하고 끝낼 시점의 타이밍을 정확히 못잡는데서 생깁니다.

![Jitter](/assets/images/jitter.jpg)

[지터는 무엇일까요?](https://roundhere.tistory.com/entry/%EC%A7%80%ED%84%B0-jitter)

또한 지터를 이렇게 설명한 것도 있습니다.

DAC 칩이라는 소자는 입력되어 온 디지털 신호를 그대로 아날로그 값으로 변환합니다. 예를 들어, ON 타이밍이 규정보다 늦고 신호가 들어가면 그만큼 ON 시간이 짧아지고 아날로그 값은 규정보다 작게 출력됩니다. 반대로 빨리 넣으면 그만큼 ON 시간이 길어져 아날로그 값이 커집니다. 이 ON-OFF의 간격이 미묘하게 흔들리는 것이 지터입니다. 

smpd에서는 이 데이터를 송출하는 타이밍을 높은 레벨로 정확하게 송출하고 있기 때문에, 소프트웨어에 의해 음질 향상이 도모되고 있다고 나는 해석하고 있습니다.

문든 든 생각이 개념상으로 보면 지터를 없애는 것은 불가능해 보입니다. ideal clock 이라는 말부터가 현실과 동떨어진 이야기죠. 

정도의 차이지만 인간이 만든 기기가 오차가 없을 수는 없다고 생각합니다. 

제가 아직도 의문을 갖는 부분은 

1) 지터 때문에 정말 음질이 나빠지는가? 2) 음질 나빠진다면 어떻게 나빠지는가? 3) 계측기가 아닌 평범한 인간이 음악 청음시 인지 가능한 수준인가 ?

입니다. 검색을 해봤지만 시원한 답을 못 찾았습니다.

*diyaudio.com 에서도 지터를 개선하면 노이즈 플로어 (Noise Floor) 개선 정도로 이야기하는 것을 본적이 있습니다.*

**smpd를 설치하고 청음하면서 느낀점이 동일 조건에서 이전 보다 소리가 더 깨끗해진 느낌을 받았지만 제가 Noise Floor 를 계측할 능력이 안되서 검증은 할 수 없습니다.** 계측이 아니라 청음으로 이를 인지하려면 적어도 주변 소음이 차단되어 있는 좋은 청음 환경이어야 하는데 안타깝게도 제 방은 그런 환경은 아닙니다.

지터 이야기는 여기까지 하겠습니다.

# DAC 의 클럭 시스템

[DAC의 마스터 클럭에 대해 ESS는 조금 바뀌었습니까?](https://nw-electric.way-nifty.com/blog/2016/02/dacess-9d4c.html)

구글의 도움을 받아서 ^^ 번역을 했습니다.

## 슬레이브 모드 (Sync)
일반적으로 DAC를 슬레이브(Sync)로 사용하는 것이 대부분입니다. 그 보다는 슬레이브 모드밖에 없는 DAC-IC가 많다고 하는 것이 옳을지도 모릅니다. TI의 PCM179x, DSD179x, 시러스의 CS439x, 아사히 화성의 AK449x 등도 예외는 아닙니다.

오실레이터가 2개 붙어 있는 것은 44.1kHz계(44.1k, 88.2k, 176.4k)와 48kHz계(48k, 96k, 192k)의 샘플링 주파수의 양쪽에 대응하기 때문입니다. (PLL등으로 어느 쪽에도 대응할 수 있는 발진원을 탑재하는 일도 있습니다.)​

재생할 악곡 데이터의 샘플링 주파수에 따라 둘 중 하나를 선택하여 마스터 클럭(시스템 클럭)에 사용합니다. 당연히 MCLK는 BCK, LRCK 등과 동기화되어야합니다.　

고급기에서는 오실레이터를 DAC-IC 가까이에 두고 MCLK를 DAC와 I2S 출력 장치 모두에 보내는 레이아웃을 취할 수 있습니다. 그 쪽이 지터가 늘어나지 않고 끝나기 때문입니다.　

## 비동기 모드 (Async)
슬레이브의 하나의 형태로, 동기하고 있지 않은 슬레이브 모드라고 하는 느낌입니다.　

ESS사의 DAC는 데이터시트가 공개되어 있지 않기 때문에 불명점이 많은 것은 확실합니다. 그래도 매니아 분들은 잘 아는 비동기 모드를 탑재하고 있습니다. 아마 ASRC(비동기 샘플링 레이트 컨버터)와 같은 것이 내장되어 있다고 생각합니다만, 자세한 것은 모릅니다.

그림으로 하면 아래와 같은 구성이 됩니다.
​
DAC측에, I2S의 입력 신호와는 관계가 없는(동기하고 있지 않은) 오실레이터를 하나 가지고 있어, 어떠한 샘플링 주파수의 데이터라도, 무심코 수신할 수 있습니다.　

ESS 이외에도 비동기 클록으로 동작하는 Maxim MAX9850이 있다. 또한 DAC-IC 자체에 ASRC를 내장하지 않아도 외부에 ASRC를 붙이면 다른 DAC에서도 마찬가지일 수 있다고 생각합니다. 고성능 ASRC로 SRC4192, AD1896, AK4137 당이 유명합니까?　

DAC-IC에 PLL이 내장되어, BCK로부터 시스템 클락을 만들어 내고 있는 PCM510x나 CS4350은, MCLK가 없어도 Sync 모드로 동작하고 있습니다.　

다음으로 DAC를 마스터로 만드는 사용법입니다.　

## 마스터 모드
이것은 「송신 디바이스」와 「DAC-IC」의 양쪽이 대응하고 있지 않으면 접속할 수 없는 조금 특수한 모드입니다.　

**유리한 점으로는 슬레이브 모드(Sync 모드)보다 MCLK, BCK의 지터가 적게 되는 부분입니다.** 같은 오실레이터를 사용해도 버퍼를 통과하거나 기판 패턴의 연장으로 수십 ps 정도의 지터가 빨리 증가합니다.　

지터 클리너를 통과하는 경우에도 MCLK를 청소할지, BCK를 청소할지, 아니면 둘 다를 청소해야 하는가. LRCK는 청소하지 않아도 될까? 등 고민할 곳이 있네요. 게다가, 지터 클리너는 대식이므로 5V나 3.3V전원의 부하가 늘어남에 의해 전원 노이즈가 늘어나거나 지터 클리너에의 배선을 늘려 버려 본말 전도는 일도 일어나지 않는다고는 할 수 없습니다.　

**마스터 모드라면 오실레이터 자신의 지터 만..... 매우 간단합니다.**　

RaspberryPi의 I2S는 슬레이브 모드일 수 있으며 SABRE 9018Q2C는 마스터 모드를 갖추고 있으므로 이번에는 이 모드를 사용할 수 있습니다.　

그래서 시작한 새로운 DAC 기판에서 「비동기 모드」와 「마스터 모드」의 2개를 듣고 비교해 보았습니다. 음질적인 부분은 정확한 것을 문장으로 전하는 것이 매우 어렵기 때문에 어디까지나 **나 개인의 감상이라고 생각해 주면 됩니다.**

### 비동기 모드 소리
따뜻함과 섬세함을 아울러 가지고 있어, 어쨌든 듣는 기분이 좋다. 평상시, 들을 수 있는 소스를 재생해도, 지금까지 눈치채지 못한 소리가 들리기 때문에 즐겁다.　

다만, 이것은 강한 칼라레이션이라고 파악할 수도 있어, 일청해 ESS사의 DAC의 소리다. 라고 알 수 있는 것 같은 생각이 든다. 특히 저음의 웜한 소리는 타사의 DAC에서는 들은 적이 없는 특징적인 기분을 느낍니다. 

>놀랍게도 이 부분은 얼마전 구매한 제 ES9038Q2M DAC 에서 느낀바와 동일합니다.^^

### 마스터 모드 소리
따뜻함과 섬세함은 있지만, 게다가 날카로움이 증가합니다.　

특징적인 저음의 웜감은 조금 감쇠해, 소스의 녹음의 차이가 잘 나오게 된다. 또, 공간의 넓이나 깊이감이 우수하고, 악기의 정위는 이쪽이 정확하게 전해 오는 것 같은 생각이 듭니다.

결국 어느 쪽이 좋은가 하는 것은 취향의 문제일지도 모릅니다. 고급 오디오로서는 소리의 전망이 좋은 마스터 모드가 어울린다고 생각합니다.

---

위의 번역은 DDC (XMOS, Combo384 등)를 사용한 구성에서의 DAC 클럭시스템의 도식도였습니다.

diyaudio.com 에서의 라즈베리파이 i2s 시스템 (PCM5122)에 대한 도식도를 가져왔습니다.

[diyAudio 라즈베리파이 i2s 시스템](https://www.diyaudio.com/forums/vendor-s-bazaar/355137-symphonic-mpd-post6242879.html)

![rpi_masterslave](/assets/images/rpi_masterslave.png)

위 다이어그램, Dac는 마스터입니다.

Dac는 모든 클록 신호를 생성합니다. 그려진 그림은 pcm5122에 대한 것입니다. dac 보드에는 마스터 클럭(MCLK)용 오실레이터가 2개 있습니다. 하나는 44.1kHz 그룹용이고 다른 하나는 48kHz용입니다. 라즈베리의 드라이버는 일부 GPIO 핀을 통해 활성화되는 것을 제어합니다. 또한 원하는 샘플 속도로 실행되도록 dac 칩을 구성합니다. 그런 다음 dac 칩은 마스터 클록을 적절한 양으로 나누어 비트 클록(BCLK)과 샘플 클록(LRCLK)을 생성합니다. Raspberry i2s 인터페이스는 슬레이브로 실행되며 BCLK에 의해 트리거될 때 DATA 라인에 다음 비트를 출력합니다. i2s fifo는 dma 컨트롤러에서 필요에 따라 채워집니다. **Pi PLL은 전혀 사용되지 않습니다.​**

하단 다이어그램, Pi는 마스터입니다.

Pi는 PLL을 사용하여 비트 클록(BCLK)과 샘플 클록(대부분의 경우 LRCLK = BCLK/64)을 생성합니다. 이 BCLK는 i2s 인터페이스를 트리거하여 위와 동일하게 다음 비트를 출력하고 dma 컨트롤러도 같은 방식으로 fifo를 채웁니다.

Dac 측에서 dac는 신호를 확인하여 전송되는 형식과 샘플 속도를 감지합니다(다이어그램은 PCM5102를 염두에 두고 그려짐). MCLK가 제공되지 않으면(RPi의 경우) **PLL(그림과 같이 별도의 장치가 아닌 dac에 내장됨)**을 사용하여 일치하는 마스터 클럭을 생성합니다.

---
라즈베리파이의 경우 위에서 설명한 도식도와 같이 라즈베리파이가 마스터가 되고 DAC가 슬레이브가 되는 방식과 그 반대가 되는 방식이 있습니다. 

알리에서 PCM5122를 탑재한 DAC를 검색하면 여러가지 제품이 검색되어 나옵니다. 같은 PCM5122 데 가격이 차이가 좀 나서 의문이셨다면 그 의문은 DAC가 Mater 를 지원하는 제품이면 Slave 보다 비쌉니다. 왜냐하면 마스터 DAC 인 경우 오실레이터와 전원부 등으로 부품과 회로가 더 복잡해 집니다. 그래서 가격이 차이가 납니다.

제가 앰만사 프리에 내장시킨 i2s DAC는 Allo Boss DAC 입니다. 녹색 NDK OSCILLATORS 라고 보일 겁니다. Master 모드를 지원하는 DAC 입니다.

![Allo Boss DAC](/assets/images/allo_boss_dac.png)

Inno-Maker 의 PCM5122 입니다. 
Allo Boss DAC 와 마찬가지로 기판에 오실레이터가 보입니다. (적색 네모) 마스터 모드 지원 DAC입니다.

![Inno Maker PCM5122](/assets/images/innomaker_pcm5122.png)

알리에서 구매한 저렴이 PCM5122 입니다. 기판을 보시면 오실레이터가 없습니다. 슬레이브 모드만 지원합니다.

![PiFi PCM5122](/assets/images/pifi_pcm5122.png)

DAC가 슬레이브 모드일 경우 철저히 라즈베리파이의 PLL 에 의존하는 구조입니다. smpd를 개발한 파팔리우스씨가 라즈베리파이의 드라이버를 새로 만든 목적이 라즈베리파이에서 생성되는 오디오용 클럭의 품질을 높이기 위한 것으로 생각됩니다.​

오늘 앰만사 프리앰프의 라즈베리파이에도 SMPD 를 설치하고 오랜만에 수퍼트라이오드를 연결해서 청음하고 있습니다. 역시 RCA 45 삼극 직열관 소리는 정말 매력적입니다.^^