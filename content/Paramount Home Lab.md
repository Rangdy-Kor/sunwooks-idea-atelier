---
publish: true
---

# Paramount Home Lab

---

## Internet

---

|    제품군     |              모델              |                                          비고                                          |
| :--------: | :--------------------------: | :----------------------------------------------------------------------------------: |
|  **라우터**   | ASUS ROG Rapture GT-BE98 Pro |                                          -                                           |
| **스위치 허브** |      QNAP QSW-M2108-2C       | NAS 연결 스위치에 Link Aggregation(LACP) 설정<br>NAS <=> Workstation은 스위치 허브 경유 없이 직결 브릿지 구성 |

## NAS

---

|      제품군      |                   모델                   |                                        비고                                         |
| :-----------: | :------------------------------------: | :-------------------------------------------------------------------------------: |
| **Main Unit** |           Synology DS1823xs+           |     가상 드라이브 마운트 시 SFTP 사용<br>DDNS 및 역방향 프록시 활용<br>외부 DSM 접속 시 QuickConnect 사용     |
|    **RAM**    |         Synology D4ES03-16G x2         |                           기존 8GB RAM 제거 후 부착 (실용량 32GB)                           |
|    **SSD**    |   Synology SNV5410 NVMe x2 800GB x2    |         RAID 1 스토리지 풀 볼륨 1 생성 (실용량 800GB)<br>DSM과 모든 Docker / DB / 패키지 설치         |
|    **HDD**    |        Synology HAT5310 20TB x6        | RAID 6 스토리지 풀 볼륨 2 구성 (실용량 80TB)<br>대용량 파일 / 미디어 저장<br>나머지 슬롯은 고장 및 점검 시 예비용으로 사용 |
|    **UPS**    |       APC Smart-UPS SMT1500RM2U        |                                         -                                         |
|  **네트워크 카드**  |           Synology E10G22-SR           |                           듀얼 포트 SFP+ 사용 (최대 대역폭 20Gbps)                           |
|   **광랜 모듈**   |  Intel 10G SFP+ SR Module (E10GSFPSR)  |                                         -                                         |
|   **광패치코드**   | Panduit LC to LC Duplex OM4 Patch Cord |                                         -                                         |

## Architecture

---

```mermaid
%%{ 
  init: { 
    'theme': 'base',
    'themeVariables': { 
      'fontSize': '13px',
      'nodeSpacing': 50,
      'rankSpacing': 40,
      'curve': 'basis'
    }
  }
}%%

flowchart TD
	ISP[**Internet ISP**] --> ROG[**Router**]

	subgraph VLAN[**논리적 네트워크 분리**]
		ROG --> Trunk["**Trunk Port**
		(VLAN 10 / 20 / 30)"]
		Trunk --> QNAP[**Switch Hub**]

	end

	subgraph Inside[**VLAN 10**<br>관리자 내부망]
		subgraph Direct["**10GbE 직결 (무경유)**"]
			PC[**Paramount Workstation**] <-- "SFP+ 광케이블" --> NAS[**NAS**]
		end
				
		subgraph Ethernet1[**10GbE 이더넷**]
			QNAP --> Access["**Access**<br>(Untagged)"]
			Access --> NAS[**NAS**]
			Access --> PC[**RUBIDIAN DESKTOP**]
		end

		subgraph SSID1[**SSID 1: 관리자 SSID**]
			WiFi --> High[**6GHz**<br>Wi-Fi 7 지원 최신 기기]
			High --> Personal[**Paramount Custom Laptop<br>Paramount Ultrabook<br>Paramount Tablet<br>Paramount Smartphone<br>Paramount Smart Watch**]
		end
	end

	subgraph Outside[**VLAN 20/30**<br>게스트 외부망]
		subgraph SSID2[**SSID 2**<br>게스트 SSID]
			Trunk --> WiFi["**2.5GbE Wi-Fi**<br>(SSID 구분)"]
			High --> NoPersonal[**기타 모바일 기기**]
			WiFi --> Low[**5GHz / 2.4GHz**<br>IoT / 게스트 기기]
		end

		subgraph Ethernet2[**10GbE 이더넷**]
			Access --> IoT[**기타 유선 장비**]
		end
	end
```
