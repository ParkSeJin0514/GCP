# 📙 09.12 GCP
## 🧩 Terraform
### 기본 구조
- tfinfra 디렉토리 생성 > mynetwork.tf, provider.tf 생성
- tfinfra 디렉토리 안에 instance 디렉토리 생성 > main.tf, variables.tf 생성

---

### provider 선언

```bash
provider "google" {}
```

- 네트워크 설정

```bash
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {          # GCP VPC 네트워크 리소스를 정의, 이름은 "mynetwork"
  name                    = "mynetwork"                  # 네트워크 이름을 "mynetwork"로 지정
  auto_create_subnetworks = true                         # GCP 기본 방식인 자동 서브넷 생성 모드 사용 (리전에 따라 자동 서브넷 생성)
}

# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {   # 방화벽 규칙 리소스 정의
  name    = "mynetwork-allow-http-ssh-rdp-icmp"                            # 방화벽 규칙 이름
  network = google_compute_network.mynetwork.self_link                     # 위에서 만든 "mynetwork" 네트워크에 적용

  allow {                                                                 # 허용 규칙 블록 1
    protocol = "tcp"                                                       # TCP 프로토콜 허용
    ports    = ["22", "80", "3389"]                                        # SSH(22), HTTP(80), RDP(3389) 포트 허용
  }

  allow {                                                                 # 허용 규칙 블록 2
    protocol = "icmp"                                                      # ICMP 허용 (핑 테스트 가능)
  }

  source_ranges = ["0.0.0.0/0"]                                            # 전 세계 모든 IP(공인 IP 포함)에서 접근 허용
}

# Create the mynet-vm-1 instance
module "mynet-vm-1" {                                                      # VM 생성 모듈 호출 (외부 모듈 : ./instance)
  source           = "./instance"                                          # ./instance 경로에 있는 모듈을 사용
  instance_name    = "mynet-vm-1"                                          # 인스턴스 이름을 "mynet-vm-1"로 지정
  instance_zone    = "asia-northeast3-a"                                   # 서울 리전 a존에 배치
  instance_network = google_compute_network.mynetwork.self_link            # 네트워크는 "mynetwork"와 연결
}

# Create the mynet-vm-2 instance
module "mynet-vm-2" {                                                      # VM 생성 모듈 호출 (외부 모듈 : ./instance)
  source           = "./instance"                                          # ./instance 경로에 있는 모듈을 사용
  instance_name    = "mynet-vm-2"                                          # 인스턴스 이름을 "mynet-vm-2"로 지정
  instance_zone    = "asia-northeast3-b"                                   # 서울 리전 b존에 배치 (다른 존에 분산 → 고가용성)
  instance_network = google_compute_network.mynetwork.self_link            # 네트워크는 "mynetwork"와 연결
}
```

### main.tf 설정

```bash
resource "google_compute_instance" "vm_instance" {   # GCP VM 인스턴스를 정의, 이름은 "vm_instance"
  name         = "${var.instance_name}"              # VM 이름, 모듈에서 넘겨받은 변수 instance_name 사용
  zone         = "${var.instance_zone}"              # VM이 배치될 존(예: asia-northeast3-a), 변수로 전달받음
  machine_type = "${var.instance_type}"              # 인스턴스 머신 타입(예 : e2-medium), 변수로 지정

  boot_disk {                                        # VM 부팅용 디스크 설정 블록
    initialize_params {                              # 디스크 초기화 파라미터
      image = "debian-cloud/debian-11"               # OS 이미지를 Debian 11로 지정 (Google 제공 퍼블릭 이미지)
    }
  }

  network_interface {                                # 네트워크 인터페이스 설정
    network = "${var.instance_network}"              # 연결할 VPC 네트워크 (외부에서 전달된 변수 사용)
    access_config {                                  # 외부 인터넷 접속을 위한 NAT IP 할당 블록
      # Allocate a one-to-one NAT IP to the instance   # 주석 : 퍼블릭 IP를 자동 할당 (없으면 외부 인터넷 불가)
    }
  }
}
```

### 변수 설정

```bash
variable "instance_name" {}          # VM 인스턴스 이름을 외부에서 입력받는 변수 (필수 입력, 기본값 없음)
variable "instance_zone" {}          # VM을 배치할 존(예 : asia-northeast3-a)을 외부에서 입력받는 변수 (기본값 없음)
variable "instance_type" {           # VM 머신 타입을 지정하는 변수
  default = "e2-micro"               # 기본값은 e2-micro (GCP 프리티어에서도 자주 쓰이는 소형 인스턴스)
}
variable "instance_network" {}       # VM이 연결될 VPC 네트워크를 입력받는 변수 (예 : google_compute_network.mynetwork.self_link)
```