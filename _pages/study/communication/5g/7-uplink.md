---
layout: posts
title: "PHY Control 시그널링 - Uplink"
permalink: /study/communication/5g/7/uplink/
description:
---

# Uplink Control Information(UCI)

Uplink Control을 위한 UCI는 3가지로 볼 수 있다. 단말이 수신한 DL-SCH 전송 블록에 대한 응답인 **HARQ Ack/Nack**, 단말이 UL-SCH 전송을 위해 Uplink 자원을 요청할 때 보내는 **Scheduling Request(SR)**, 단말이 기지국으로 보내는 Downlink 채널에 대한 채널 상태 보고(**CSI Report**)가 그것이다.

<img class="modal" src="/_pages/study/communication/5g/images/7/uplink/1.png" alt="<b>[Fig. 1]</b> Types of UCI, (a) HARQ Ack/Nack (b) Scheduling Request (c) CSI Report <a href='#Reference'>[1]</a>."/>

UCI는 **PUCCH 또는 PUSCH**를 통해 전송된다. DCI의 경우에는 오로지 PDCCH를 통해서만 전송이 되었는데, 단말의 성능이 제한되어 있기 때문에 이미 할당받은 Uplink 자원이 있어서 PUSCH를 보낼 경우에는 UCI를 PUSCH를 통해 전송을 하고, 그 외에는 PUCCH를 통해 전송한다. PUSCH로 보내는 UCI에는 SR을 제외한 HARQ Ack/Nack, CSI Report가 해당이 되는데, PUSCH를 보낸다는 말은 이미 스케줄링이 되어있다는 것이기 때문에 SR을 보낼 필요가 없다.


# PUCCH

PUCCH를 위한 자원할당은 Payload 크기와 채널 상태에 따라 적합한 **PUCCH format**을 선택하여 이루어지게 된다.

<div class="post__stage-container">
    <div class="post__stage">
        <img class="modal" src="/_pages/study/communication/5g/images/7/uplink/2.png" alt="<b>[Fig. 2]</b> PUCCH formats <a href='#Reference'>[2]</a>."/>
    </div>
    <div class="post__stage">
        <img class="modal" src="/_pages/study/communication/5g/images/7/uplink/6.png" alt="<b>[Fig. 3]</b> PUCCH formats <a href='#Reference'>[3]</a>."/>
    </div>
</div>

**2 bits 이하의 payload**의 경우 PUCCH format 0과 1을 사용하게 된다. HARQ Ack/Nack이나 SR을 보낼 때 사용된다. CSI Report는 보통 bit 수가 많기 때문에(2 bits 초과) PUCCH format 2~4를 사용해 보낸다. PUCCH format 0은 OFDM 심볼 최대 2개까지만 사용할 수 때문에 채널 상태가 좋지 않거나 먼 거리에 있을 경우에는 OFDM 심볼 4~14개를 사용하는 PUCCH format 1을 사용한다.

**Payload가 2 bits를 초과**하는 경우 PUCCH format 2와 3을 사용하는데 마찬가지로 채널 상태가 좋지 않거나 먼 거리에 있을 경우에는 PUCCH format 3를 사용한다. OFDM 심볼을 최대 2개까지만 사용하는 PUCCH format 0, 2를 **short PUCCH format**이라고 하고 심볼 4~14개를 사용하는 PUCCH format 1, 3, 4를 **long PUCCH format**이라고 한다.

PUCCH format 2와 3은 2 bits를 초과하는 payload에 대한 UE multiplexing을 지원하지 않는다. 그에 대한 대안으로 PUCCH format 4는 FDM을 통해 4대 이상의 UE multiplexing을 지원한다.

<div class="post__stage-container">
    <div class="post__stage">
        <img class="modal" src="/_pages/study/communication/5g/images/7/uplink/3.png" alt="<b>[Fig. 4]</b> General description of NR PUCCH <a href='#Reference'>[4]</a>."/>
    </div>
    <div class="post__stage">
        <img class="modal" src="/_pages/study/communication/5g/images/7/uplink/4.png" alt="<b>[Fig. 5]</b> PUCCH formats <a href='#Reference'>[4]</a>."/>
    </div>
</div>

Long PUCCH format은 frequency hopping이 적용돼 할당된 대역폭 양 극단에 위치하고, short PUCCH format은 보통 슬롯의 마지막에 전송되는데 꼭 그런건 아니고 저지연이 중요한 경우나 스케줄링 요청이 빈번한 경우, 더 큰 서브캐리어 간격이 필요한 경우에는 다른 위치에서 보낼 수 있다.

PUCCH format 별로 리소스 매핑 과정이 다르다.

## PUCCH format 0

우선 PUCCH format 0의 기본 시퀀스로는 base [시퀀스](/study/communication/5g/2/2/)를 사용한다. 이 기본 시퀀스에 12가지의 위상 회전(phase rotation)에 정보를 싣는다. 즉 base 시퀀스 자체에는 정보가 없는데 이 시퀀스에 서로 다른 12개의 orthogonal한 직교 시퀀스가 존재해서 정보에 의해 시퀀스의 위상이 결정된다. 이를 'cyclic shift'라고 부른다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/7.png" alt="<b>[Fig. 6]</b> HARQ와 SR을 이용한 위상 회전의 예 <a href='#Reference'>[5]</a>."/>

12개의 위상 회전 중에서 HACK Ack/Nack이 1 bit일 경우에는 위상 차이가 $2 \pi \cdot 6/12$이고, 2 bits일 경우 $2 \pi \cdot 3/12$인 것이다.

다수의 단말이 같은 시간-주파수 자원을 사용할 경우 서로 다른 기준 위상 회전을 가질 수 있다. 예를 들어 한 단말은 0, $2 \pi \cdot 6/12$를 사용하고, 다른 단말은 $2 \pi \cdot 3/12$ 및 $2 \pi \cdot 9/12$를 사용할 수 있다.

위상 오프셋의 경우 슬롯마다 호핑되도록 설정할 수 있는데, 오프셋의 경우 pseudo random 시퀀스에 의해 결정된다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/5_0.png" alt="<b>[Fig. 6]</b> PUCCH format 0 <a href='#Reference'>[5]</a>."/>

## PUCCH format 1

PUCCH는 4~14 심볼을 이용해 2 bit 까지 전송하는데, control information용 OFDM 심볼과 reference signal용 심볼이 분리되어 있다. CI 심볼과 RS 심볼을 반반 정도 배치하는게 좋은 절충선이라 본다.

1 bit 일 때는 BPSK, 2 bits 일 때는 QPSK를 사용하는데 변조된 신호에 PUCCH format 0에서와 같이 12 길이의 low-PAPR 시퀀스가 곱해지고 sequnece hopping이 적용된다. 그 후 orthogonal DFT 코드가 곱해진다. 직교 코드를 곱하는 것은 다수의 단말이 같은 기준 시퀀스와 위상을 사용할 때도 코드를 통해 분리하겠다는 것이다.

추가로 long PUCCH format의 경우 frequency hopping을 적용할 수 있다고 했는데, hopping을 할지의 여부는 PUCCH 리소스 설정에 의해 결정되고 hopping 위치는 심볼 길이에 의해 결정된다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/5_1.png" alt="<b>[Fig. 7]</b> PUCCH format 1 <a href='#Reference'>[5]</a>."/>

## PUCCH format 2

PUCCH format 2부터는 2 bits 보다 큰 payload를 가지는 CSI Report나 많은 수의 HARQ Ack/Nack, 또는 두 가지를 동시에 보낼 때의 경우인데 인코딩 해야 할 비트가 너무 크면 CSI Report는 누락시킬 수 있다.

Payload가 크기 때문에 CRC와 채널 코딩 과정을 거친다. 무조건 CRC를 붙이는 건 아니고 payload가 큰 경우에 붙인다. CRC 포함 11 bits 이하면 Reed-Muller를 사용하고 11 bits보다 많으면 polar 코딩을 사용한다. 이후 scramblig 과정이 있는데, scrambling sequence는 C-RNTI와 Cell ID를 이용해 생성한다<sup><a href='#Reference'>[7]</a></sup>.

$$
\begin{align}
\widetilde{b}(i) &= (b(i) + c(i)) \, \text{mod} \, 2 \quad &(1) \\
c_{\text{init}} &= n_{\text{RNTI}} \cdot 2^{15} + n_{\text{ID}} \quad &(2)
\end{align}
$$

그 뒤에는 마찬가지로 QPSK 변조 과정을 거친다. PUCCH format 2부터는 RB의 개수를 가변적으로 사용하는데, 이 개수는 payload 크기와 code rate의 최곳값에 의해 결정된다. PUCCH format 2도 format 0과 마찬가지로 일반적으로는 슬롯의 맨 뒤에 전송되지만, 필요한 경우 바뀔 수 있다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/5_2.png" alt="<b>[Fig. 8]</b> PUCCH format 2 <a href='#Reference'>[5]</a>."/>

## PUCCH format 3

PUCCH format 3는 할당되는 리소스가 가장 많은 format이다. 전송 구조는 format 2와 거의 동일하고, modulation의 경우 QPSK가 default로 사용되지만 cubic metric을 낮추기 위해 π/2-BPSK를 사용할 수 있고, OFDM 심볼에 매핑 시 DFT-precoding을 적용할 수 있다. 중요한 정보일수록 DMRS에 가깝게 매핑시킨다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/5_3.png" alt="<b>[Fig. 9]</b> PUCCH format 3 <a href='#Reference'>[5]</a>."/>

## PUCCH format 4

전반적인 구조는 format 3와 동일하나 format 4의 경우는 말했듯이 다수의 단말에 multiplexing을 하기 위해 하나의 resource block을 사용한다.

<img class="modal img__small" src="/_pages/study/communication/5g/images/7/uplink/5_4.png" alt="<b>[Fig. 10]</b> PUCCH format 4 <a href='#Reference'>[5]</a>."/>

## PUCCH resource set




---

# <a name="Reference"></a>Reference

1. “5G Physical Uplink Control Channel (PUCCH) and Uplink Control Information (UCI).” Youtube, MATLAB, 9 Sept. 2019, [https://www.youtube.com/watch?v=Tc_ECMWSH30](https://www.youtube.com/watch?v=Tc_ECMWSH30){:target="_blank"}.
2. “PUCCH Formats in 5G.” Youtube, Wireless Explained, 4 May 2023, [https://www.youtube.com/watch?v=g6IjHANJ180](https://www.youtube.com/watch?v=g6IjHANJ180){:target="_blank"}.
3. MathWorks, "5G NR Uplink with PUCCH Vector Waveform Generation," [Online]. Available: [https://kr.mathworks.com/help/5g/ug/uplink-with-pucch-carrier-waveform-generation.html](https://kr.mathworks.com/help/5g/ug/uplink-with-pucch-carrier-waveform-generation.html){:target="_blank"}. [Accessed: 26- Feb- 2024].
4. NTT DOCOMO, INC., “WI Summary of New Radio Access Technology,” [3GPP TDocs (written contributions) at meeting](https://www.3gpp.org/dynareport?code=TDocExMtg--RP-80--18663.htm){:target="_blank"}, RP-180990, 2018.
5. Erik Dahlman, Stefan Parkvall, Johan Sköld,Chapter 10 - Physical-Layer Control Signaling, Editor(s): Erik Dahlman, Stefan Parkvall, Johan Sköld, 5G NR (Second Edition), Academic Press, 2021, Pages 197-241.
6. ShareTechnote, [5G/NR - PUCCH](https://www.sharetechnote.com/html/5G/5G_PUCCH.html){:target="_blank"}.
7. 3GPP, "[TS 38.211 v17.6.0](https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3213){:target="_blank"}".
{:.post__reference}