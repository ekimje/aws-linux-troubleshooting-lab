# Terraform 완전 입문 — 개념, 문법, 구조

ㅍSep 28, 2026 · @Someone

## 1. Terraform이 무엇인가

Terraform은 **"서버와 네트워크를 어떻게 만들지"를 글(코드)로 적어두면, 그대로 클라우드에 만들어주는 도구**다. 이런 방식을 IaC(Infrastructure as Code, 코드로 관리하는 인프라)라고 부른다.

### 비유: 인테리어 설계도

집 인테리어를 한다고 해보자.

- **콘솔로 직접 만들기** = 내가 직접 가구점에 가서 소파 사고, 벽지 고르고, 하나씩 배치하는 것. 끝나고 나면 "내가 뭘 어떻게 했더라?"가 기록에 안 남는다. 똑같은 집을 하나 더 꾸미려면 처음부터 다시 기억을 더듬어야 한다.
- **Terraform** = 설계도를 그려서 시공업체에 넘기는 것. "거실에 3인용 소파 1개, 벽지는 흰색"이라고 적어두면 업체가 그대로 만든다. 설계도가 남으니 똑같은 집을 몇 번이든 다시 만들 수 있고, 무엇이 바뀌었는지도 설계도의 수정 이력으로 알 수 있다.

여기서 **설계도 = `.tf` 파일**, **시공업체 = Terraform**, **집 = AWS에 만들어진 실제 자원**이다.

### 선언형: "어떻게"가 아니라 "무엇을" 적는다

Terraform의 가장 중요한 성격은 \*\*선언형(declarative)\*\*이라는 것이다.

- **절차형(명령형):** "서버를 만들어라. 그다음 방화벽을 열어라. 그다음…" — 순서대로 할 일을 적는다. 셸 스크립트, Ansible 플레이북이 여기에 가깝다.
- **선언형:** "최종적으로 서버 2대와 방화벽 규칙 1개가 **있어야 한다**" — 원하는 결과 상태만 적는다. 순서와 방법은 Terraform이 알아서 정한다.

선언형이라서 생기는 좋은 점이 있다. 같은 코드를 10번 실행해도 결과는 똑같다. 이미 서버 2대가 있으면 Terraform은 "이미 원하는 상태네" 하고 아무것도 하지 않는다. 이 성질을 \*\*멱등성(idempotency)\*\*이라고 한다.

### 왜 쓰는가

- **재현성:** `apply` 한 번에 전체 환경이 생기고, `destroy` 한 번에 전부 사라진다. 크레딧을 아끼면서 실습할 때 특히 유용하다.
- **기록과 협업:** 코드가 Git에 남으니 누가 언제 무엇을 바꿨는지 추적된다.
- **미리보기:** 실제로 바꾸기 전에 `plan`으로 "무엇이 만들어지고, 바뀌고, 지워지는지"를 먼저 확인할 수 있다.
- **멀티 클라우드:** 같은 문법으로 AWS, Azure, GCP 등을 다룬다. 쓰는 리소스 이름만 달라진다.

### Terraform이 하지 않는 일

Terraform은 **"자원이 존재하도록"** 만드는 도구다. 서버 **안**에 프로그램을 설치하고 설정 파일을 고치는 일은 주력이 아니다. 그건 Ansible 같은 구성 관리 도구의 몫이다. 이 프로젝트에서 Terraform은 집을 짓고, Ansible은 그 집에 가구를 들이고 관리한다고 생각하면 된다.

## 2. 핵심 개념 7가지

Terraform 코드는 거의 전부 아래 7가지 조합이다. 이 7개만 확실히 알면 남의 코드를 읽을 수 있다.

| 개념 | 한 줄 설명 | 인테리어 비유 |
| --- | --- | --- |
| provider | 어느 클라우드와 대화할지 정하는 연결 플러그인 | 어느 시공업체와 계약할지 |
| resource | 새로 **만들** 자원 1개 | 설계도의 "소파 1개 설치" 항목 |
| data | 이미 **있는** 자원을 조회만 함 | 건물에 원래 있던 창문 크기 확인 |
| variable | 밖에서 넣어주는 입력값 | 고객이 고르는 벽지 색 |
| output | 다 만든 뒤 알려주는 결과값 | 완공 후 받는 "현관 비밀번호" 안내 |
| locals | 코드 안에서 반복되는 값에 붙인 별명 | 설계도 상단의 "기본 색상 = 흰색" 메모 |
| state | 실제로 무엇을 만들었는지 적어둔 장부 | 업체가 보관하는 시공 완료 기록 |

### 2-1. provider (공급자)

Terraform 본체는 AWS를 모른다. AWS와 대화하는 법은 **AWS provider**라는 플러그인이 알고 있다. 그래서 가장 먼저 "AWS provider를 쓰겠다, 서울 리전에서"라고 선언한다.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"   # 어디서 받아올지 (공식 레지스트리)
      version = "~> 5.0"          # 5.x 버전대만 허용
    }
  }
}

provider "aws" {
  region = "ap-northeast-2"      # 서울 리전
}
```

- `version = "~> 5.0"`은 "5.0 이상, 6.0 미만"이라는 뜻이다. 버전을 고정하지 않으면 어느 날 provider가 크게 바뀌어 코드가 깨질 수 있다.
- AWS 로그인 정보(액세스 키)는 **코드에 절대 쓰지 않는다.** 로컬 PC에서 `aws configure`로 설정해두면 provider가 알아서 읽어간다.

### 2-2. resource (리소스)

**Terraform이 만들고, 관리하고, 지울 자원 하나**를 뜻한다. 가장 많이 쓰는 블록이다.

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

이 한 덩어리를 읽는 법:

- `resource` — "만들 자원이다"라는 키워드
- `"aws_vpc"` — **리소스 타입**. "AWS의 VPC"라는 종류. 앞부분 `aws_`가 provider 이름이다
- `"main"` — **내가 붙이는 이름(로컬 이름)**. AWS에는 전달되지 않고, 코드 안에서 이 자원을 부를 때만 쓴다
- `cidr_block = ...` — **인자(argument)**. 이 VPC의 설정값

이 자원을 다른 곳에서 부를 때는 `aws_vpc.main`이라고 쓴다. 타입 + 점 + 이름이 이 자원의 **주소**다.

### 2-3. data (데이터 소스)

resource는 "만든다", data는 "**있는 걸 찾아본다**". 예를 들어 EC2를 만들려면 OS 이미지(AMI) ID가 필요한데, AMI는 AWS가 이미 만들어둔 것이라 내가 만들 수 없다. 이럴 때 data로 "최신 Amazon Linux 이미지 ID를 찾아줘"라고 조회한다.

```hcl
data "aws_ami" "al2023" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

부를 때는 앞에 `data.`가 붙는다: `data.aws_ami.al2023.id`

### 2-4. variable (입력 변수)

코드에 값을 직접 박아두지 않고, **밖에서 바꿔 넣을 수 있게** 구멍을 뚫어두는 것이다. 함수의 매개변수와 같다.

```hcl
variable "my_ip" {
  description = "SSH를 허용할 내 공인 IP (예: 1.2.3.4/32)"
  type        = string
}
```

부를 때는 `var.my_ip`. 값을 넣는 방법은 여러 가지다.

- `terraform.tfvars` 파일에 `my_ip = "1.2.3.4/32"`라고 적기 (가장 흔함)
- 실행할 때 `terraform apply -var="my_ip=1.2.3.4/32"`
- 아무것도 안 주면 실행 중에 Terraform이 물어본다
- `default = "..."`를 적어두면 기본값으로 쓰인다

내 IP처럼 **공개 저장소에 올리면 안 되는 값**은 `terraform.tfvars`에 넣고 이 파일을 `.gitignore`에 추가한다.

### 2-5. output (출력값)

`apply`가 끝난 뒤 화면에 보여줄 값이다. 만들어지기 전엔 알 수 없는 값(EC2의 IP 같은)을 확인할 때 쓴다.

```hcl
output "control_public_ip" {
  value = aws_instance.control.public_ip
}
```

이 프로젝트에서는 output으로 나온 IP를 Ansible 인벤토리에 넣는 데 쓴다. 나중에 `terraform output`으로 다시 볼 수도 있다.

### 2-6. locals (지역 값)

여러 곳에서 반복되는 값에 **별명**을 붙인다. variable과 달리 밖에서 바꿀 수 없고, 코드 안에서만 쓰는 계산값이다.

```hcl
locals {
  project = "ransomware-lab"
  common_tags = {
    Project = local.project
    Owner   = "jiyun"
  }
}
```

부를 때는 `local.project` (정의는 `locals`, 부를 때는 `local` — s가 없다는 점 주의).

### 2-7. state (상태 파일)

Terraform이 \*\*"내가 무엇을 만들었고, 그게 AWS에서 어떤 ID인지"\*\*를 기록해두는 장부다. `terraform.tfstate`라는 JSON 파일로 저장된다. 코드(원하는 상태)와 state(내가 아는 현재 상태)를 비교해서 무엇을 해야 할지 결정한다. 워낙 중요해서 6장에서 따로 자세히 다룬다.

## 3. 작업 흐름: init → plan → apply → destroy

Terraform 작업은 항상 같은 순서를 따른다. 가장 중요한 습관은 **plan 결과를 읽지 않고 apply하지 않는 것**이다.

&#91;embedded content: Terraform 작업 흐름 · 4단계 + destroy\]

plan은 코드와 state를 비교해 할 일을 계산하고, apply는 그 결과를 state에 기록한다. destroy는 state에 적힌 것만 지운다.

### 명령어별 설명

- **`terraform init`** — 프로젝트 폴더에서 처음 한 번 실행한다. 코드에 적힌 provider(AWS 플러그인)를 내려받아 `.terraform/` 폴더에 저장하고, 버전을 `.terraform.lock.hcl`에 기록한다. provider를 추가하거나 버전을 바꾸면 다시 실행한다.
- **`terraform fmt`** — 들여쓰기와 `=` 정렬을 자동으로 맞춰준다. 커밋 전에 습관처럼 실행.
- **`terraform validate`** — 문법 오류, 없는 인자 이름 같은 걸 AWS에 접속하지 않고 검사한다.
- **`terraform plan`** — "지금 apply하면 무슨 일이 일어날지"를 보여준다. 아무것도 바꾸지 않는다.
- **`terraform apply`** — plan을 다시 보여주고 `yes`를 입력하면 실제로 만든다. 완료되면 state를 갱신한다.
- **`terraform destroy`** — state에 있는 자원을 전부 지운다. 콘솔에서 직접 만든 자원은 state에 없으니 지우지 않는다.
- **`terraform output`** / **`terraform state list`** — 결과값 다시 보기 / 관리 중인 자원 목록 보기.

### plan 결과 읽는 법

plan 출력의 맨 앞 기호가 핵심이다.

| 기호 | 뜻 | 주의할 점 |
| --- | --- | --- |
| `+` | 새로 만듦 (create) | 예상한 것만 있는지 확인 |
| `~` | 제자리에서 수정 (update in-place) | 보통 안전 |
| `-` | 삭제 (destroy) | 의도한 건지 반드시 확인 |
| `-/+` | 지우고 다시 만듦 (replace) | **가장 위험.** EC2면 안의 데이터가 사라지고 IP가 바뀜 |

마지막 줄 `Plan: 3 to add, 1 to change, 0 to destroy.`에서 숫자를 확인한다. 특히 destroy가 0이 아니면 멈추고 이유를 찾는다. 출력에 `# forces replacement`가 붙은 줄이 있으면 그 인자를 바꿔서 교체가 일어난다는 뜻이다. `(known after apply)`는 "만들어봐야 알 수 있는 값(ID, IP 등)"이라는 뜻이다.

## 4. HCL 문법 읽고 쓰는 법

Terraform 코드는 HCL(HashiCorp Configuration Language)이라는 언어로 쓴다. 문법은 **블록, 인자, 표현식** 세 가지뿐이라 프로그래밍 언어보다 훨씬 단순하다.

### 4-1. 블록과 인자

```hcl
블록종류 "라벨1" "라벨2" {
  인자이름 = 값

  중첩블록 {
    인자이름 = 값
  }
}
```

- **블록:** 중괄호 `{ }`로 묶인 덩어리. `resource`, `variable`, `provider` 등이 블록 종류다. 라벨 개수는 블록 종류마다 정해져 있다 (resource는 2개, variable은 1개, locals는 0개).
- **인자:** `이름 = 값` 형태. 한 블록 안에서 같은 인자를 두 번 쓸 수 없다.
- **중첩 블록:** 블록 안의 블록. `=` 없이 쓴다. 같은 중첩 블록을 여러 번 쓸 수 있다 (예: `filter { }` 여러 개).
- **주석:** `#` 또는 `//` 한 줄, `/* */` 여러 줄.

헷갈리는 포인트: `tags = { ... }`는 `=`가 있으니 인자(값이 맵)이고, `filter { ... }`는 `=`가 없으니 **중첩 블록**이다. 어느 쪽인지는 공식 문서의 리소스 페이지에 적혀 있다.

### 4-2. 값의 종류 (타입)

| 타입 | 예 | 설명 |
| --- | --- | --- |
| string | `"10.0.0.0/16"` | 문자열. 반드시 큰따옴표 |
| number | `8`, `0.5` | 숫자. 따옴표 없음 |
| bool | `true`, `false` | 참/거짓. 따옴표 없음 |
| list | `["a", "b"]` | 순서 있는 목록. 0번부터 셈 |
| map | `{ Name = "lab", Env = "dev" }` | 키-값 묶음 |
| object | `{ cidr = "10.0.1.0/24", public = true }` | 키마다 타입이 다른 묶음 |

`"true"`(문자열)와 `true`(bool)는 다르다. 따옴표 하나로 에러가 날 수 있다.

### 4-3. 참조: 다른 값을 가리키는 법

| 쓰는 법 | 가리키는 것 |
| --- | --- |
| `aws_vpc.main.id` | 내가 만든 리소스의 속성 (타입.이름.속성) |
| `data.aws_ami.al2023.id` | 데이터 소스의 속성 |
| `var.my_ip` | 입력 변수 |
| `local.common_tags` | locals 값 |
| `aws_subnet.private[0].id` | count로 만든 리소스의 0번째 |
| `aws_subnet.private["a"].id` | for\_each로 만든 리소스의 "a" 키 |

속성(attribute)이란 리소스가 만들어진 뒤 생기는 값이다. `id`, `arn`, `public_ip` 등이 있고, 각 리소스 문서 하단의 "Attribute Reference"에 목록이 있다. 내가 적은 인자(`cidr_block` 등)도 그대로 참조할 수 있다.

### 4-4. 문자열 안에 값 넣기

```hcl
tags = {
  Name = "${local.project}-vpc"    # → "ransomware-lab-vpc"
}
```

`${ }` 안에 참조를 넣으면 문자열에 값이 끼워진다. 참조만 단독으로 쓸 때는 `${ }` 없이 `vpc_id = aws_vpc.main.id`처럼 쓴다. 예전 코드의 `"${aws_vpc.main.id}"`는 옛날 문법이니 따라 하지 않는다.

### 4-5. 자주 쓰는 함수

| 함수 | 예 | 결과 |
| --- | --- | --- |
| `merge` | `merge(local.common_tags, { Name = "vpc" })` | 두 맵을 합침 (태그에 자주 씀) |
| `cidrsubnet` | `cidrsubnet("10.0.0.0/16", 8, 1)` | `"10.0.1.0/24"` — 큰 대역을 잘게 나눔 |
| `length` | `length(["a", "b"])` | `2` |
| `file` | `file("scripts/setup.sh")` | 파일 내용을 문자열로 읽음 |
| `lookup` | `lookup(var.sizes, "dev", "t3.micro")` | 맵에서 찾고 없으면 기본값 |

`cidrsubnet("10.0.0.0/16", 8, 1)` 해석: /16에 8비트를 더해 /24로 쪼개고, 그중 1번째 조각을 가져온다. 0번은 `10.0.0.0/24`, 1번은 `10.0.1.0/24`. 함수는 `terraform console`을 실행해서 직접 쳐보며 연습할 수 있다.

### 4-6. 같은 걸 여러 개 만들기: count와 for\_each

**count** — 숫자만큼 복사한다. `count.index`로 몇 번째인지 안다.

```hcl
resource "aws_subnet" "private" {
  count      = 2
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet("10.0.0.0/16", 8, count.index + 10)
}
# → aws_subnet.private[0], aws_subnet.private[1]
```

**for\_each** — 맵이나 집합의 **키마다** 하나씩 만든다.

```hcl
resource "aws_subnet" "this" {
  for_each   = { public = "10.0.1.0/24", private = "10.0.2.0/24" }
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  tags       = { Name = each.key }
}
# → aws_subnet.this["public"], aws_subnet.this["private"]
```

**언제 무엇을?** 목록 중간 것을 지우면 count는 번호가 당겨지면서 뒤의 자원까지 교체되는 사고가 난다. for\_each는 이름(키)으로 관리해서 그런 일이 없다. **서로 설정이 다르면 for\_each, 완전히 똑같은 걸 N개면 count**가 원칙이다. 이 프로젝트처럼 자원이 몇 개 안 되면 그냥 리소스 블록을 따로 쓰는 게 가장 읽기 쉽다. 처음에는 count/for\_each 없이 짜보고, 반복이 거슬릴 때 도입해도 늦지 않다.

### 4-7. 조건식

```hcl
instance_type = var.env == "prod" ? "t3.medium" : "t3.micro"
```

`조건 ? 참일 때 : 거짓일 때`. `count = var.create_nat ? 1 : 0`처럼 "켜고 끄기 스위치"로 자주 쓴다. NAT 게이트웨이를 실습 안 할 때 꺼두는 데 응용할 수 있다.

## 5. 의존성: 생성 순서는 누가 정하나

Terraform에서는 **파일 순서나 블록 순서가 실행 순서가 아니다.** 참조 관계를 보고 Terraform이 스스로 순서를 정한다.

### 5-1. 암묵적 의존성 (대부분 이걸로 충분)

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "private" {
  vpc_id     = aws_vpc.main.id      # ← VPC를 참조
  cidr_block = "10.0.2.0/24"
}
```

서브넷이 `aws_vpc.main.id`를 참조하므로, Terraform은 "서브넷을 만들려면 VPC ID가 필요하다 → VPC 먼저"라고 판단한다. 서브넷 블록을 VPC 블록보다 위에 적어도, 다른 파일에 적어도 결과는 같다. 지울 때는 반대로 서브넷을 먼저 지우고 VPC를 나중에 지운다.

서로 참조하지 않는 자원들은 **동시에(병렬로)** 만든다. 그래서 apply가 생각보다 빠르다.

### 5-2. 명시적 의존성: depends\_on

참조는 없지만 순서가 필요한 경우가 가끔 있다. 대표적인 예가 **NAT 게이트웨이와 인터넷 게이트웨이**다. NAT는 인터넷 게이트웨이가 붙은 VPC에서만 제대로 동작하지만, NAT 코드에는 IGW를 참조하는 인자가 없다.

```hcl
resource "aws_nat_gateway" "this" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  depends_on = [aws_internet_gateway.this]   # IGW 먼저 만들어라
}
```

`depends_on`은 꼭 필요할 때만 쓴다. 남발하면 병렬 생성이 막혀 느려지고, 코드가 왜 그런 순서인지 읽기 어려워진다. 먼저 "참조로 연결할 수 없나?"를 생각한다.

### 5-3. 의존성 그래프 확인하기

`terraform graph` 명령은 자원 간 연결을 그래프 형식(DOT)으로 출력한다. 온라인 Graphviz 뷰어에 붙여넣으면 그림으로 볼 수 있다. 내가 의도한 순서대로 연결됐는지 확인할 때 쓴다.

### 5-4. 순환 참조 (Cycle 에러)

A가 B를 참조하고 B가 A를 참조하면 Terraform은 무엇을 먼저 만들지 결정할 수 없어 `Error: Cycle`을 낸다. 보안그룹 두 개가 서로를 규칙 안에서 참조할 때 자주 생긴다. 해결법은 **규칙을 보안그룹 블록 밖으로 분리**하는 것이다. 이 프로젝트에서 규칙을 `aws_vpc_security_group_ingress_rule` 같은 별도 리소스로 쓰라고 한 이유 중 하나가 이것이다. SG 두 개가 먼저 만들어지고, 규칙은 그다음에 둘을 연결한다.

## 6. State 깊게 보기

state는 Terraform의 **기억**이다. state를 잃으면 Terraform은 AWS에 무엇이 있는지 모르는 상태가 되고, 같은 코드로 apply하면 자원을 **또 하나** 만들려고 한다. 그래서 초보자가 가장 많이 사고를 내는 곳이 state다.

### 6-1. 왜 state가 필요한가

코드에는 `resource "aws_vpc" "main"`이라고만 적혀 있다. 그런데 AWS에서 이 VPC의 실제 이름은 `vpc-0a1b2c3d...` 같은 ID다. "코드의 `aws_vpc.main` = AWS의 `vpc-0a1b2c3d`"라는 **연결표**가 state에 저장된다. 이 연결표가 있어야 Terraform이 "이건 이미 만든 거니까 수정만 하면 되겠다"라고 판단할 수 있다.

plan이 하는 일을 정확히 말하면 이렇다.

1. state를 읽어서 관리 중인 자원 목록과 ID를 확인한다
2. AWS에 실제로 물어봐서 현재 상태를 새로 고친다 (refresh)
3. 코드에 적힌 원하는 상태와 비교한다
4. 차이를 메우기 위한 작업 목록(+, \~, -)을 만든다

### 6-2. drift: 코드 밖에서 바꾼 경우

Terraform으로 만든 보안그룹 규칙을 누군가 AWS 콘솔에서 직접 수정했다고 하자. 이렇게 코드와 실제가 어긋난 상태를 drift(표류)라고 한다. 다음 plan 때 Terraform은 그 변경을 발견하고, **코드대로 되돌리겠다**고 제안한다. 코드가 기준(정답)이기 때문이다.

교훈: Terraform으로 만든 자원은 **Terraform으로만 수정한다.** 콘솔은 보기 전용으로 생각한다.

이 프로젝트에서 중요한 포인트가 여기 있다. 사고 대응 중에 Ansible이 타깃의 보안그룹을 격리 SG로 바꾸면, 그 순간 Terraform 입장에서는 drift가 생긴다. 그래서 복구 플레이북이 끝날 때 **원래 SG로 되돌리는 단계**가 반드시 있어야 한다. 그래야 다음 plan이 깨끗하다.

### 6-3. 로컬 state vs 원격 state

기본적으로 state는 코드 폴더에 `terraform.tfstate` 파일로 생긴다(로컬 state). 혼자 PC 한 대에서 작업하면 이걸로 충분하다. 문제는 다음과 같다.

- PC가 고장 나거나 폴더를 지우면 state가 사라진다
- 다른 PC나 다른 사람과 같이 작업할 수 없다
- 두 사람이 동시에 apply하면 state가 꼬인다

그래서 실무에서는 state를 **S3 버킷**에 저장한다(원격 state, backend). 설정은 이렇게 한다.

```hcl
terraform {
  backend "s3" {
    bucket       = "jiyun-tfstate-bucket"   # 미리 만들어둔 S3 버킷
    key          = "ransomware-lab/terraform.tfstate"
    region       = "ap-northeast-2"
    use_lockfile = true                     # 동시 실행 방지 잠금
  }
}
```

- backend 블록에는 `var.`나 `local.`을 쓸 수 없다. 값을 직접 적는다.
- state 버킷 자체는 이 코드로 만들지 않는다. 닭이 먼저냐 달걀이 먼저냐 문제가 생기기 때문이다. 콘솔에서 따로 만들거나 별도 Terraform 폴더로 만든다.
- 버킷에 \*\*버전 관리(versioning)\*\*를 켜두면 state가 망가져도 이전 버전으로 되돌릴 수 있다.
- 예전 자료에는 잠금용으로 DynamoDB 테이블을 쓰는 방식이 많이 나온다. 최근 Terraform 버전은 S3만으로 잠글 수 있는 `use_lockfile`을 지원하니, 설치한 Terraform 버전의 문서를 확인하고 쓴다.

**이 프로젝트에서는?** 혼자 작업하니 로컬 state로 시작해도 된다. 0단계에서 원격 state로 해두면 "실무 방식을 적용했다"고 README에 쓸 수 있어 포트폴리오 점수가 올라간다. 선택은 자유.

### 6-4. state 보안: 절대 Git에 올리지 않는다

state 파일은 **평문 JSON**이다. 자원 ID, IP뿐 아니라 경우에 따라 비밀번호 같은 민감한 값이 그대로 들어간다. 공개 저장소를 쓰고 있으니 `.gitignore`에 반드시 넣는다.

```
# .gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
crash.log
*.pem
```

반대로 `.terraform.lock.hcl`은 **올리는 게 맞다.** provider 버전을 고정해서 어느 PC에서든 같은 버전을 쓰게 해주기 때문이다.

### 6-5. state 관련 명령어

| 명령 | 용도 |
| --- | --- |
| `terraform state list` | 관리 중인 자원 목록 |
| `terraform state show aws_vpc.main` | 특정 자원의 상세 정보 (ID, 속성) |
| `terraform import` / `import` 블록 | 콘솔에서 만든 기존 자원을 state에 편입 |
| `terraform state rm 주소` | 자원은 AWS에 두고 state에서만 뺌 |
| `terraform plan -refresh-only` | drift만 확인 (코드 변경 없이) |

`state rm`과 state 파일 직접 편집은 위험하다. 무엇을 하는지 확실할 때만 쓰고, 그 전에 state 파일을 백업해둔다.

## 7. 파일과 폴더 구조

Terraform은 **한 폴더 안의 모든 `.tf` 파일을 하나로 합쳐서** 읽는다. 파일을 어떻게 나누든 결과는 같다. 그러니 파일 나누기는 순전히 **사람이 읽기 편하게** 하는 작업이다.

### 7-1. 이 프로젝트 추천 구조

```
terraform/
├── versions.tf       # terraform 블록: 필요한 Terraform·provider 버전, backend
├── providers.tf      # provider "aws" 블록 (리전, 기본 태그)
├── variables.tf      # 모든 variable 블록
├── locals.tf         # 공통 태그, 이름 접두사
├── network.tf        # VPC, 서브넷, IGW, NAT, EIP, 라우팅 테이블
├── security.tf       # 보안그룹과 규칙 (정상/격리)
├── iam.tf            # 역할, 정책, 인스턴스 프로파일
├── compute.tf        # AMI 조회(data), 키 페어, EC2 2대, 데이터 EBS
├── backup.tf         # DLM 스냅샷 정책 (3단계에서 추가)
├── monitoring.tf     # CloudWatch 알람, SNS (4단계에서 추가)
├── outputs.tf        # 모든 output 블록
├── terraform.tfvars  # 내 변수 값 (Git 제외)
└── terraform.tfvars.example  # 변수 예시 (Git에 올림)
```

### 7-2. 나누는 원칙

- **역할(AWS 서비스 영역)별로 나눈다.** "네트워크 문제면 network.tf만 보면 된다"가 되도록.
- **variable, output은 한 파일에 모은다.** 이 모듈에 무엇을 넣고 무엇이 나오는지 한눈에 보이게.
- **tfvars.example 파일을 둔다.** 실제 값은 숨기되, 다른 사람이 저장소를 받았을 때 어떤 값을 채워야 하는지 알 수 있게 한다. 포트폴리오를 보는 사람에 대한 배려이기도 하다.
- **Terraform 명령은 이 폴더 안에서 실행한다.** 폴더 하나가 state 하나와 짝이다.

### 7-3. 모듈이란 (지금은 몰라도 되지만 알아두기)

**모듈**은 여러 리소스를 묶어서 재사용하는 단위다. 사실 지금 만드는 `terraform/` 폴더 자체도 모듈(루트 모듈)이다. 다른 폴더를 불러 쓰면 그게 자식 모듈이다.

```hcl
module "vpc" {
  source = "./modules/vpc"     # 내가 만든 모듈 폴더
  cidr   = "10.0.0.0/16"       # 모듈의 variable에 값 전달
}
# 모듈의 output은 module.vpc.vpc_id 처럼 꺼내 씀
```

공식 레지스트리에는 `terraform-aws-modules/vpc/aws` 같은 유명 모듈도 있다. 몇 줄이면 VPC 전체가 생긴다. 하지만 **공부 목적이라면 모듈 없이 리소스를 하나씩 직접 쓰는 걸 추천한다.** 모듈은 편하지만 안에서 무엇이 만들어지는지 가려져서, 면접에서 "NAT를 왜 public 서브넷에 두나요?" 같은 질문에 답하기 어려워진다. 직접 다 짜본 뒤에 리팩터링 단계로 모듈화하면 좋은 공부가 된다.

### 7-4. 코드 스타일 규칙

- 리소스 로컬 이름은 소문자와 밑줄: `aws_instance.control_node` (하이픈 X)
- 리소스가 하나뿐이면 `this` 또는 `main`이라는 이름을 흔히 쓴다
- 로컬 이름에 타입을 반복하지 않는다: `aws_vpc.main` O / `aws_vpc.main_vpc` X
- 모든 variable에 `description`과 `type`을 적는다
- 모든 자원에 태그를 붙인다. provider 블록의 `default_tags`를 쓰면 한 번에 적용된다

```hcl
provider "aws" {
  region = var.region
  default_tags {
    tags = {
      Project   = "ransomware-lab"
      ManagedBy = "terraform"
    }
  }
}
```

태그를 붙여두면 AWS 콘솔의 비용 탐색기에서 이 프로젝트에 크레딧이 얼마나 나갔는지 따로 볼 수 있다.

## 8. AWS 네트워크 개념과 Terraform 리소스

직접 짤 때 막히는 건 보통 Terraform 문법이 아니라 **AWS 네트워크 개념**이다. 이 장은 개념과 리소스 이름, 필수 인자만 연결해둔다. 코드는 직접 짜보는 부분으로 남겨둔다.

&#91;embedded content: 이 프로젝트 네트워크 · public 1개 + private 1개\]

타깃이 패키지를 받을 때 트래픽은 타깃 → NAT → IGW → 인터넷 순서로 나간다. 밖에서 타깃으로 먼저 들어오는 길은 없고, 접속은 컨트롤 노드를 거쳐서만 한다.

### 8-1. 비유: 아파트 단지

- **VPC** = 아파트 단지 전체. 담장으로 둘러싸인 나만의 네트워크
- **서브넷** = 단지 안의 동(棟). 동마다 호수(IP) 범위가 정해져 있음
- **IGW** = 단지 정문. 바깥 도로(인터넷)와 연결
- **라우팅 테이블** = 동마다 붙은 길 안내표. "밖에 나갈 때는 정문으로" 또는 "경비실을 거쳐서"
- **NAT** = 경비실 대리 배송. private 동 주민은 직접 밖에 못 나가고, 경비실이 대신 물건을 받아다 준다. 밖에서는 경비실 주소만 보인다
- **보안그룹** = 각 집 현관문의 출입 명단. 누가 들어올 수 있는지 집마다 정함

### 8-2. 개념별 리소스와 핵심 인자

| 개념 | Terraform 리소스 | 핵심 인자 | 알아야 할 것 |
| --- | --- | --- | --- |
| VPC | `aws_vpc` | `cidr_block`, `enable_dns_hostnames` | DNS 옵션을 켜야 인스턴스가 호스트 이름을 얻음 |
| 서브넷 | `aws_subnet` | `vpc_id`, `cidr_block`, `availability_zone`, `map_public_ip_on_launch` | 서브넷은 AZ(가용 영역) 하나에 속함. public만 공인 IP 자동 부여 |
| 인터넷 게이트웨이 | `aws_internet_gateway` | `vpc_id` | VPC당 1개 |
| 탄력적 IP | `aws_eip` | `domain = "vpc"` | NAT에 붙일 고정 공인 IP |
| NAT 게이트웨이 | `aws_nat_gateway` | `allocation_id`, `subnet_id` | **public** 서브넷에 둔다. 켜져 있는 동안 계속 과금 |
| 라우팅 테이블 | `aws_route_table` | `vpc_id`, `route { cidr_block, gateway_id / nat_gateway_id }` | public은 IGW로, private은 NAT로 |
| 라우팅 연결 | `aws_route_table_association` | `subnet_id`, `route_table_id` | 이걸 빼먹으면 라우팅 테이블이 서브넷에 적용 안 됨 |
| 보안그룹 | `aws_security_group` | `name`, `vpc_id` | `vpc_id` 없으면 default VPC에 생김 |
| SG 인바운드 규칙 | `aws_vpc_security_group_ingress_rule` | `security_group_id`, `ip_protocol`, `from_port`, `to_port`, `cidr_ipv4` 또는 `referenced_security_group_id` | 소스를 IP 대신 다른 SG로 지정 가능 |
| SG 아웃바운드 규칙 | `aws_vpc_security_group_egress_rule` | 위와 동일 | Terraform은 기본 egress를 제거하므로 직접 작성 |

### 8-3. 개념 보충

**CIDR 읽는 법** — `10.0.0.0/16`에서 `/16`은 앞의 16비트(10.0)가 고정이라는 뜻이다. 뒤로 올 수 있는 주소는 약 6만 5천 개. `/24`는 앞 24비트(10.0.1)가 고정이라 256개다. AWS는 서브넷마다 5개를 예약해서 실제로는 251개를 쓴다. 서브넷 대역은 VPC 대역 안에 있어야 하고, 서로 겹치면 안 된다.

**public / private의 진짜 의미** — AWS에는 "public 서브넷"이라는 설정 칸이 없다. 연결된 라우팅 테이블에 IGW로 가는 길이 있으면 public, 없으면 private이다. 이름은 그냥 사람이 붙이는 것이다.

**라우팅 테이블의 local 경로** — 모든 라우팅 테이블에는 `10.0.0.0/16 → local`이 자동으로 들어간다. 그래서 같은 VPC 안의 서버끼리는 서브넷이 달라도 별도 설정 없이 통신된다. 컨트롤 노드가 타깃에 사설 IP로 SSH할 수 있는 이유다. 단, 보안그룹은 따로 열어줘야 한다.

**보안그룹은 stateful** — 들어오는 요청을 허용하면 그에 대한 응답은 egress 규칙과 상관없이 자동으로 나간다. 반대도 마찬가지다. 그래서 "SSH 인바운드만 열어도 SSH가 된다". 보안그룹은 허용 규칙만 있고, 막는 규칙은 없다. 명단에 없으면 전부 차단이다.

**EC2 리소스 핵심 인자** — `aws_instance`는 `ami`, `instance_type`, `subnet_id`, `vpc_security_group_ids`(리스트), `key_name`, `iam_instance_profile`, `associate_public_ip_address`를 주로 쓴다. 보안그룹 인자는 `security_groups`가 아니라 `vpc_security_group_ids`를 쓴다. 앞의 것은 올드 방식이라 VPC에서 쓰면 인스턴스가 자꾸 교체되는 문제가 생긴다.

**데이터 EBS 볼륨** — 루트 볼륨과 분리하려면 `aws_ebs_volume`(볼륨 자체, 인스턴스와 **같은 AZ**여야 함) + `aws_volume_attachment`(어느 인스턴스에 어떤 장치 이름으로 붙일지)를 따로 만든다. 복구 때 Ansible이 이 볼륨을 바꿔 끼우므로, 이 부분도 6-2의 drift 이야기가 적용된다. 복구 후 Terraform이 새 볼륨을 모르는 문제는 5단계에서 따로 다루면 된다.

## 9. 자주 하는 실수와 에러 읽는 법

에러 메시지는 겁먹을 대상이 아니라 **위치와 원인을 알려주는 안내문**이다. 읽는 순서만 알면 대부분 혼자 해결된다.

### 9-1. 에러 메시지 읽는 순서

```
Error: creating EC2 Instance: InvalidParameterCombination: ...

  with aws_instance.target,
  on compute.tf line 12, in resource "aws_instance" "target":
  12: resource "aws_instance" "target" {
```

1. **`Error:` 다음 첫 문장** — 무엇을 하다가 실패했는지 (여기선 EC2 생성)
2. **AWS 에러 코드** — `InvalidParameterCombination`, `UnauthorizedOperation` 같은 단어. 이걸 그대로 검색하면 원인이 나온다
3. **`with` / `on ... line`** — 어느 리소스, 어느 파일 몇 번째 줄인지

에러가 여러 개면 **맨 위부터** 해결한다. 뒤의 에러는 앞 에러 때문에 연쇄로 생긴 경우가 많다.

### 9-2. 자주 보는 에러

| 에러 (일부) | 원인 | 해결 |
| --- | --- | --- |
| `Reference to undeclared resource` | 오타, 또는 없는 리소스 이름 참조 | 타입.이름 철자 확인 |
| `Unsupported argument` | 그 리소스에 없는 인자 이름 | 공식 문서의 Argument Reference 확인 |
| `Missing required argument` | 필수 인자 빠뜨림 | 문서에서 Required 표시된 인자 추가 |
| `Error: Cycle` | 서로 참조하는 순환 | 5-4 참고. 규칙을 별도 리소스로 분리 |
| `No valid credential sources found` | AWS 로그인 정보 없음 | `aws configure` 또는 `aws sts get-caller-identity`로 확인 |
| `UnauthorizedOperation` | IAM 권한 부족 | 사용하는 IAM 사용자 권한 확인 |
| `security group and subnet belong to different networks` | SG에 `vpc_id` 누락 → default VPC에 생성됨 | SG에 `vpc_id` 추가 |
| `InvalidAMIID.NotFound` | 다른 리전의 AMI ID를 붙여넣음 | AMI는 리전마다 다름. data 소스로 조회 |
| `Error acquiring the state lock` | 이전 실행이 비정상 종료돼 잠금이 남음 | 다른 실행이 정말 없는지 확인 후 `terraform force-unlock <ID>` |
| `Inconsistent dependency lock file` | provider를 바꾸고 init을 안 함 | `terraform init -upgrade` |
| `DependencyViolation` (destroy 중) | 콘솔에서 만든 자원이 VPC 안에 남아 있음 | 콘솔에서 그 자원 먼저 삭제 |

### 9-3. 초보가 자주 하는 실수

- **plan을 안 읽고 apply** — 특히 `-/+`(교체)와 destroy 숫자를 확인하지 않아 서버가 통째로 다시 만들어진다.
- **콘솔과 Terraform을 섞어 쓰기** — 콘솔에서 수정하면 drift, 콘솔에서 만든 자원은 destroy가 못 지운다. 크레딧이 새는 대표 원인이다.
- **state 파일 삭제** — 폴더를 정리하다 tfstate를 지우면 AWS에 자원이 남아 있는데 Terraform은 모른다. 요금은 계속 나간다. 이때는 콘솔에서 직접 지우거나 import해야 한다.
- **tfstate, tfvars, 키 파일을 Git에 올림** — 공개 저장소면 즉시 노출된다. 올렸다면 파일 삭제만으로는 부족하다. 이력에 남으므로 키를 폐기하고 새로 발급한다.
- **리전 착각** — provider는 서울인데 콘솔은 버지니아를 보고 있으면 "아무것도 안 만들어졌다"고 오해한다. 콘솔 오른쪽 위 리전을 확인한다.
- **destroy 후 확인 안 함** — destroy가 중간에 실패하면 일부가 남는다. 마지막 줄 `Destroy complete! Resources: N destroyed.`를 확인하고, `terraform state list`가 비어 있는지 본다.
- **NAT 켜두고 잊기** — 쓰지 않을 땐 destroy. 4-7의 스위치 변수로 NAT만 끄는 방법도 있다.

### 9-4. 막힐 때 확인 순서

1. `terraform fmt` → `terraform validate`로 문법부터 거른다
2. 에러 코드를 공식 문서와 검색으로 확인한다
3. `terraform state list` / `state show`로 Terraform이 아는 것과 콘솔에 있는 것을 비교한다
4. 그래도 모르겠으면 `TF_LOG=DEBUG terraform apply`로 상세 로그를 본다 (Windows PowerShell은 `$env:TF_LOG="DEBUG"`)
5. AI에 물어볼 때는 **에러 전문 + 해당 리소스 코드 + 무엇을 하려 했는지**를 같이 준다. 이것만 지켜도 답의 질이 확 달라진다

## 10. Ansible과 연결하기

Terraform의 일은 **서버가 켜지는 순간** 끝나고, Ansible의 일은 **서버에 접속하는 순간** 시작된다. 둘을 잇는 것은 딱 세 가지 정보다: 어디에 접속할지(IP), 무엇으로 접속할지(SSH 키), 어떤 역할의 서버인지(태그).

### 10-1. 전체 실행 순서

1. 로컬 PC에서 `terraform apply` → VPC, EC2 2대, SG 등 생성
2. Terraform이 컨트롤 노드에 Ansible 설치 (user\_data, 10-4 참고)
3. 컨트롤 노드에 SSH 접속, 저장소를 받음 (`git clone`)
4. 컨트롤 노드에서 인벤토리 확인: `ansible-inventory --graph`
5. `ansible-playbook site.yml` → 타깃에 Docker 설치, 서비스 배포
6. 사고 시 `ansible-playbook recover.yml`

### 10-2. 방법 A: output을 보고 인벤토리를 손으로 작성

가장 단순한 방법이다. Terraform output으로 타깃의 사설 IP를 출력하고, 그걸 인벤토리 파일에 적는다.

```hcl
# outputs.tf
output "target_private_ip" {
  value = aws_instance.target.private_ip
}
```

```ini
# ansible/inventory/hosts.ini
[target]
10.0.2.15 ansible_user=ec2-user
```

**단점:** destroy 후 다시 만들면 IP가 바뀌어 매번 고쳐야 한다. 공부 첫 단계에서만 쓴다.

### 10-3. 방법 B: 동적 인벤토리 (추천)

Ansible의 `amazon.aws.aws_ec2` 인벤토리 플러그인은 **AWS에 직접 물어봐서** 서버 목록을 만든다. Terraform은 태그만 잘 붙여두면 된다. IP가 바뀌어도 아무것도 고칠 필요가 없다. 실무에서 가장 많이 쓰는 방식이다.

Terraform 쪽 — 역할 태그를 붙인다:

```hcl
resource "aws_instance" "target" {
  # ...
  tags = {
    Name = "lab-target"
    Role = "target"
  }
}
```

Ansible 쪽 — 파일 이름이 반드시 `aws_ec2.yml` 또는 `aws_ec2.yaml`로 끝나야 한다:

```yaml
# ansible/inventory/aws_ec2.yml
plugin: amazon.aws.aws_ec2
regions:
  - ap-northeast-2
filters:
  tag:Project: ransomware-lab
  instance-state-name: running
keyed_groups:
  - key: tags.Role          # Role 태그 값으로 그룹 생성
    prefix: role            # → role_target 그룹
compose:
  ansible_host: private_ip_address   # VPC 안에서 사설 IP로 접속
```

플레이북에서는 `hosts: role_target`으로 대상을 고른다.

**준비물:**

- 컨트롤 노드에 `ansible-galaxy collection install amazon.aws`와 Python `boto3` 설치
- 컨트롤 노드 IAM 역할에 `ec2:DescribeInstances` 권한 (서버 목록 조회용). 액세스 키를 서버에 두지 않고 역할로 해결하는 게 핵심이다

**이 구조의 장점:** 복구 플레이북도 같은 컬렉션(amazon.aws)의 모듈로 스냅샷 조회, 볼륨 생성·연결, 보안그룹 변경을 한다. 인벤토리부터 복구까지 AWS 인증 방식이 IAM 역할 하나로 통일된다.

### 10-4. user\_data: 부팅 시 한 번만 도는 스크립트

EC2의 `user_data`는 **첫 부팅 때 한 번** 실행되는 셸 스크립트다. 컨트롤 노드에 Ansible을 깔아두는 정도의 "최소 준비"에 적당하다.

```hcl
resource "aws_instance" "control" {
  # ...
  user_data = file("${path.module}/scripts/control_init.sh")
}
```

```bash
#!/bin/bash
# scripts/control_init.sh — Amazon Linux 2023 기준
dnf install -y python3-pip git
pip3 install ansible boto3
```

주의할 점:

- 실행 결과는 콘솔에 안 보인다. 서버 안의 `/var/log/cloud-init-output.log`로 확인한다
- 스크립트를 수정해도 이미 떠 있는 서버에서 다시 실행되지 않는다. 기본 설정에서는 plan에 `~`로 나오지만 재실행은 안 된다. 새로 돌리려면 `user_data_replace_on_change = true`로 서버를 교체해야 한다
- 설정이 길어지면 user\_data가 아니라 Ansible로 옮긴다. "설치는 user\_data, 설정과 배포는 Ansible"로 나눈다

### 10-5. provisioner는 쓰지 않는다

Terraform에는 `local-exec`, `remote-exec`처럼 apply 중에 명령을 실행하는 provisioner가 있다. 인터넷 예제에서 `local-exec`로 ansible-playbook을 부르는 코드를 자주 보게 된다. 하지만 HashiCorp 공식 문서도 **최후의 수단**으로 분류한다. 실패해도 state에 기록이 애매하게 남고, 다시 실행되지 않기 때문이다. 이 프로젝트처럼 **Terraform 따로, Ansible 따로 실행**하는 게 정석이고, 면접에서 설명하기도 깔끔하다.

## 11. Docker와 연결하기

Terraform과 Docker를 잇는 방법은 세 가지가 있다. 이 프로젝트에서는 **Docker 설치와 컨테이너 실행은 Ansible에 맡기는 방식**을 쓴다. 나머지 둘은 개념으로 알아두면 된다.

| 방법 | Terraform이 하는 일 | 언제 쓰나 |
| --- | --- | --- |
| A. Ansible에 맡김 (이 프로젝트) | EC2만 만들고 끝 | 서버 안 설정이 복잡하거나 재실행이 필요할 때 |
| B. user\_data로 설치 | 부팅 시 Docker 설치 스크립트 전달 | 설정이 짧고 한 번이면 될 때 |
| C. Docker provider | 컨테이너·이미지·네트워크 자체를 리소스로 관리 | Docker 호스트가 이미 있고 컨테이너만 선언형으로 관리할 때 |

### 11-1. 방법 A: Ansible이 Docker를 담당 (선택한 방식)

복구 흐름에 "컨테이너 중지 → 볼륨 교체 → 재기동"이 들어 있다. 이건 **사고 때마다 순서대로 실행하는 절차**라서 Ansible의 영역이다. 역할 분담을 정리하면:

- **Terraform:** 타깃 EC2, 데이터 EBS 볼륨, SG
- **Ansible (site.yml):** Docker 엔진 설치, 데이터 볼륨 마운트, `docker-compose.yml` 복사, 서비스 기동
- **Ansible (recover.yml):** `community.docker.docker_compose_v2` 모듈로 서비스 중지와 재기동

컨테이너 데이터는 반드시 **데이터 EBS 볼륨을 마운트한 경로**에 두도록 compose 파일에서 바인드 마운트한다. 그래야 볼륨만 스냅샷으로 교체해도 서비스 데이터가 복원된다. 이 연결이 이 프로젝트 설계의 핵심 고리다.

### 11-2. 방법 B: user\_data로 Docker 설치

```bash
#!/bin/bash
# Amazon Linux 2023 기준
dnf install -y docker
systemctl enable --now docker
usermod -aG docker ec2-user
```

간단하지만 10-4에서 본 한계(한 번만 실행, 결과가 안 보임)가 그대로 있다. "Docker 설치까지만 user\_data, 서비스 배포는 Ansible"처럼 섞어 쓰는 것도 흔하다. 이 프로젝트에서는 Ansible 쪽에 모아두는 게 README 설명이 더 깔끔하다.

### 11-3. 방법 C: Terraform Docker provider

Terraform에는 Docker를 직접 다루는 provider(`kreuzwerker/docker`)도 있다. provider만 바뀔 뿐, 2장의 resource 개념이 그대로 적용된다는 걸 보여주는 좋은 예다.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}

provider "docker" {}   # 로컬 Docker 엔진에 연결

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "web" {
  name  = "web"
  image = docker_image.nginx.image_id
  ports {
    internal = 80
    external = 8080
  }
}
```

`terraform apply` 하면 nginx 컨테이너가 뜨고, `destroy` 하면 사라진다. **AWS 비용 없이 PC에서 Terraform 문법을 연습하기에 가장 좋다.** Docker Desktop이 깔린 PC라면 과제 1 전에 이걸로 init → plan → apply → destroy 흐름을 먼저 익혀도 된다.

다만 운영 서버의 컨테이너를 이 방식으로 관리하지 않는 이유가 있다. 원격 Docker 호스트에 접속하려면 Docker API를 네트워크로 노출해야 하고, 컨테이너 재시작 같은 일상 작업마다 state와 어긋나기 쉽다. 컨테이너 운영은 보통 Ansible, Compose, 또는 ECS·Kubernetes 같은 오케스트레이터의 몫이다.

### 11-4. 참고: ECR (AWS의 이미지 저장소)

직접 만든 이미지를 AWS에 보관하려면 `aws_ecr_repository` 리소스로 저장소를 만든다. 이 프로젝트는 Docker Hub 공식 이미지로 충분해서 쓰지 않는다. private 서브넷에서 NAT 없이 ECR을 쓰려면 VPC 엔드포인트가 필요하고 추가 요금이 붙는다는 점만 기억해두면 된다.

## 12. Azure로 확장하기

Azure도 Terraform으로 똑같이 다룬다. **문법, 명령어, state, 의존성은 전부 같고 provider와 리소스 이름만 바뀐다.** 1\~9장을 이해했다면 Azure는 "단어장만 새로 외우는" 수준이다.

### 12-1. azurerm provider 설정

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}                          # 비어 있어도 반드시 있어야 하는 블록
  subscription_id = var.subscription_id
}
```

- 로그인은 AWS의 `aws configure` 대신 \*\*Azure CLI의 `az login`\*\*으로 한다. provider가 그 로그인 정보를 읽어간다.
- `features {}`를 빼면 에러가 난다. 초보가 가장 먼저 만나는 Azure 에러다.
- 4.x 버전부터는 구독 ID를 provider에 명시해야 한다. 구독 ID는 `az account show`로 확인하고, tfvars에 넣는다.

### 12-2. Azure에만 있는 개념: 리소스 그룹

Azure의 모든 자원은 반드시 **리소스 그룹**이라는 폴더 안에 들어가야 한다. 그래서 Azure Terraform 코드는 거의 항상 리소스 그룹 생성으로 시작하고, 이후 모든 리소스가 이걸 참조한다.

```hcl
resource "azurerm_resource_group" "lab" {
  name     = "rg-ransomware-lab"
  location = "Korea Central"
}

resource "azurerm_virtual_network" "main" {
  name                = "vnet-lab"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
}
```

두 번째 블록이 첫 번째를 참조하므로 리소스 그룹이 먼저 만들어진다(5장의 암묵적 의존성). 실습이 끝나면 리소스 그룹째로 지우면 안의 자원이 전부 정리되는 것도 장점이다. 물론 Terraform으로 만들었다면 `destroy`를 쓴다.

### 12-3. AWS ↔ Azure 개념 대응표

| 개념 | AWS 리소스 | Azure 리소스 | 차이점 |
| --- | --- | --- | --- |
| 자원 묶음 | (없음, 태그로 구분) | `azurerm_resource_group` | Azure는 필수 |
| 가상 네트워크 | `aws_vpc` | `azurerm_virtual_network` | 이름만 다름 (VNet) |
| 서브넷 | `aws_subnet` | `azurerm_subnet` | Azure 서브넷은 AZ에 묶이지 않음 |
| 방화벽 규칙 | `aws_security_group` | `azurerm_network_security_group` | NSG는 **차단 규칙과 우선순위 번호**가 있음 |
| 방화벽 연결 | EC2의 `vpc_security_group_ids` | `azurerm_subnet_network_security_group_association` 등 | NSG는 서브넷이나 NIC 단위로 붙임 |
| 인터넷 출구 | IGW + 라우팅 | 기본 제공 (별도 리소스 불필요) | Azure VM은 기본적으로 외부로 나갈 수 있었으나, 정책이 바뀌는 추세라 문서 확인 필요 |
| NAT | `aws_nat_gateway` | `azurerm_nat_gateway` | 개념 동일 |
| 공인 IP | `aws_eip` | `azurerm_public_ip` |  |
| 가상 머신 | `aws_instance` | `azurerm_linux_virtual_machine` + `azurerm_network_interface` | Azure는 NIC를 별도 리소스로 먼저 만듦 |
| 추가 디스크 | `aws_ebs_volume` | `azurerm_managed_disk` |  |
| 스냅샷 | `aws_ebs_snapshot`, DLM | `azurerm_snapshot`, Azure Backup |  |
| 서버 권한 | IAM 역할 + 인스턴스 프로파일 | Managed Identity + 역할 할당 | 액세스 키 없이 권한 부여하는 같은 발상 |
| 모니터링 | CloudWatch | Azure Monitor |  |

**가장 헷갈리는 차이 두 가지:**

- **NSG는 규칙마다 우선순위(priority) 숫자가 있다.** 100\~4096 사이이고 **숫자가 작을수록 먼저** 적용된다. AWS SG에는 없는 "Deny(차단)" 규칙도 있다. 그래서 순서를 잘못 잡으면 허용한 줄 알았던 트래픽이 막힌다.
- **VM은 NIC(네트워크 카드)가 따로다.** AWS는 EC2에 서브넷을 바로 지정하지만, Azure는 NIC를 먼저 만들어 서브넷에 붙이고, VM이 그 NIC를 참조한다.

### 12-4. 한 코드에서 AWS와 Azure를 같이 쓰기

`required_providers`에 둘 다 적고 provider 블록을 두 개 두면 **한 번의 apply로 두 클라우드에 자원을 만들 수 있다.** 각 리소스는 이름 앞부분(`aws_`, `azurerm_`)으로 어느 provider를 쓸지 자동으로 결정된다.

```hcl
terraform {
  required_providers {
    aws     = { source = "hashicorp/aws", version = "~> 5.0" }
    azurerm = { source = "hashicorp/azurerm", version = "~> 4.0" }
  }
}
```

**이 프로젝트에 응용한다면?** 본 프로젝트가 끝난 뒤의 확장 아이디어로만 두는 게 좋다. 예를 들어 "AWS 스냅샷과 별도로 Azure Blob Storage에 백업 사본을 두는 멀티 클라우드 백업"은 랜섬웨어 대응의 3-2-1 백업 원칙(사본 3개, 매체 2종, 1개는 다른 장소)과 연결해 설명할 수 있다. 다만 두 클라우드의 인증과 비용을 동시에 관리해야 하니, AWS 버전을 완성하고 수치까지 뽑은 다음에 도전한다.

### 12-5. Ansible과 Docker도 Azure에서 그대로

- **Ansible:** `azure.azcollection` 컬렉션에 AWS의 `aws_ec2`와 같은 역할의 동적 인벤토리 플러그인(`azure_rm`)과 디스크·스냅샷 모듈이 있다. 인벤토리 파일 이름은 `azure_rm.yml`로 끝나야 한다.
- **Docker:** VM 안에서 돌아가는 건 똑같다. 11장의 방식이 클라우드와 상관없이 그대로 통한다. 이게 "서버 안은 Ansible + Docker, 서버 밖은 Terraform"으로 나눈 설계의 장점이다. 클라우드를 바꿔도 Terraform 코드만 다시 쓰면 된다.

## 13. 직접 짜보기: 단계별 과제

한 번에 다 짜지 말고, **작게 만들고 → apply → 확인 → destroy**를 반복한다. 과제마다 끝나면 plan 결과가 예상과 같은지 스스로 말로 설명해본다. 설명이 안 되면 그 부분이 아직 모르는 부분이다.

### 과제 1. 첫 apply (VPC 하나)

- [ ] `versions.tf`, `providers.tf` 작성 → `terraform init` 성공
- [ ] `aws_vpc` 하나만 만들고 plan에 `1 to add`가 나오는지 확인
- [ ] apply 후 콘솔에서 VPC 확인, `terraform state list`로 목록 확인
- [ ] `cidr_block`을 바꿔 plan → `-/+`(교체)가 나오는 것 관찰 (apply는 안 해도 됨)
- [ ] destroy

**스스로 답해보기:** init이 만든 `.terraform/` 폴더와 lock 파일은 각각 무엇인가? 왜 cidr\_block 변경은 교체인가?

### 과제 2. 변수와 출력

- [ ] `region`, `project`, `my_ip`를 variable로 빼고 `terraform.tfvars`에 값 넣기
- [ ] `default_tags`로 모든 자원에 태그 붙이기
- [ ] VPC ID를 output으로 출력
- [ ] `.gitignore` 작성, `terraform.tfvars.example` 만들기

**스스로 답해보기:** variable과 locals의 차이는? tfvars를 왜 Git에 안 올리나?

### 과제 3. public 네트워크

- [ ] public 서브넷, IGW, 라우팅 테이블(0.0.0.0/0 → IGW), 라우팅 연결
- [ ] 컨트롤 노드 SG: SSH를 `var.my_ip`에서만 허용, egress 직접 작성
- [ ] data 소스로 AMI 조회, 키 페어 연결, 컨트롤 노드 EC2 생성
- [ ] output의 공인 IP로 SSH 접속 성공

**스스로 답해보기:** 라우팅 연결을 빼면 어떻게 되나? egress 규칙을 빼면 무엇이 안 되나? (직접 빼고 확인해보면 가장 잘 기억된다)

### 과제 4. private 네트워크와 NAT

- [ ] private 서브넷, EIP, NAT 게이트웨이(public 서브넷에), private 라우팅 테이블(→ NAT)
- [ ] 타깃 SG: SSH를 컨트롤 노드 **SG에서만** 허용 (`referenced_security_group_id`)
- [ ] 타깃 EC2 생성 (공인 IP 없음)
- [ ] 컨트롤 노드 → 타깃 사설 IP로 SSH 성공, 타깃에서 `curl https://example.com` 성공
- [ ] 확인 끝나면 바로 destroy (NAT 요금)

**스스로 답해보기:** NAT를 private 서브넷에 두면 왜 안 되나? 타깃 SG의 소스를 IP가 아니라 SG로 한 이유는?

### 과제 5. IAM과 데이터 볼륨

- [ ] 컨트롤 노드 역할: EC2·스냅샷·SG 조작 권한 / 타깃 역할: CloudWatch 메트릭 전송 권한
- [ ] 인스턴스 프로파일로 각 EC2에 연결
- [ ] 데이터 EBS 볼륨 + attachment (타깃과 같은 AZ)
- [ ] 타깃에서 `lsblk`로 볼륨이 보이는지 확인

**스스로 답해보기:** 역할(role), 정책(policy), 인스턴스 프로파일은 각각 무엇이고 왜 세 개로 나뉘나?

### 최종 셀프 체크리스트

면접에서 이 질문들에 막힘없이 답할 수 있으면 "개념은 안다"고 말할 수 있다.

- [ ] Terraform은 선언형이다. 이게 무슨 뜻이고 Ansible과 어떻게 역할을 나눴나?
- [ ] state는 무엇이고, 잃어버리면 어떻게 되나? 원격 state는 왜 쓰나?
- [ ] plan에서 `~`와 `-/+`의 차이는? 교체가 위험한 이유는?
- [ ] Terraform은 생성 순서를 어떻게 정하나? `depends_on`은 언제 쓰나?
- [ ] count와 for\_each의 차이는?
- [ ] public 서브넷과 private 서브넷을 가르는 기준은?
- [ ] 보안그룹이 stateful이라는 건 무슨 뜻인가?
- [ ] drift란 무엇이고, 이 프로젝트에서는 어디서 생길 수 있나?
- [ ] 공개 저장소에 올리면 안 되는 파일은 무엇이고 왜인가?

### 공부 자료

- Terraform 공식 튜토리얼 "Get Started - AWS" (developer.hashicorp.com): 이 문서의 1\~6장을 손으로 따라 해볼 수 있다
- Terraform Registry의 AWS provider 문서: 리소스마다 Example Usage, Argument Reference, Attribute Reference 세 부분만 읽으면 된다. 코드를 짤 때 가장 많이 열게 될 페이지다
- `terraform console`: 함수와 참조를 실험하는 놀이터
