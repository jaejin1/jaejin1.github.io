# aws vpc peering using terraform


Terraform을 공부하면서 AWS의 VPC를 생성해보고, peering까지 해보자.

<!--more-->

us-east-1, us-east-2 region을 연결 해보고자 한다.

~~~terraform
terraform {
  required_version = ">= 0.10.3"
}


provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  region = "us-east-2"
  alias = "us-east-2"
}
~~~

2개의 region을 이용하기 위해 provider를 정의 해준다.

~~~terraform
resource "aws_vpc" "create_vpc" {
    cidr_block = "10.10.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support = true
    instance_tenancy = "default"

    tags {
        Name = "jaejin test"
    }
}

resource "aws_vpc" "create_vpc_2" {
    provider = "aws.us-east-2"
    cidr_block = "172.10.0.0/16"
    enable_dns_hostnames = true
    enable_dns_support = true
    instance_tenancy = "default"

    tags {
        Name = "jaejin test222"
    }
}
~~~

연결 시킬 VPC를 region 마다 생성한다.

~~~terraform
resource "aws_default_route_table" "create_vpc" {
    default_route_table_id = "${aws_vpc.create_vpc.default_route_table_id}"

    tags {
        Name = "default"
    }
}

resource "aws_default_route_table" "create_vpc_2" {
    provider = "aws.us-east-2"

    default_route_table_id = "${aws_vpc.create_vpc_2.default_route_table_id}"

    tags {
        Name = "default"
    }
}
~~~

생성한 VPC의 라우팅 테이블을 생성해준다.
추후에 VPC들을 바라보게 만들고, 외부에서 ssh 접속을 위해 외부 접속도 받게 한다.

~~~terraform
resource "aws_subnet" "subnet_1" {
    vpc_id = "${aws_vpc.create_vpc.id}"
    cidr_block = "10.10.1.0/24"
    map_public_ip_on_launch = true
    availability_zone = "${data.aws_availability_zones.available.names[0]}"
    tags = {
        Name = "public-az-1"
    }
}


resource "aws_subnet" "subnet_2" {
    provider = "aws.us-east-2"

    vpc_id = "${aws_vpc.create_vpc_2.id}"
    cidr_block = "172.10.1.0/24"
    map_public_ip_on_launch = true
    availability_zone = "${data.aws_availability_zones.available_2.names[0]}"
    tags = {
        Name = "public-az-2"
    }
}

data "aws_availability_zones" "available" {}

data "aws_availability_zones" "available_2" {
    provider = "aws.us-east-2"
}

~~~


라우팅 테이블에 다른 region의 VPC를 등록하기 위해 subnet을 생성한다. 
subnet을 등록할때 availability zone을 불러와서 등록 하면된다. list로 받아오기 때문에 0번째 인덱스만 사용하도록 설정해 놓았다.

즉 여러 availability zone을 이용할 수 있다.

~~~terraform

resource "aws_internet_gateway" "igw_1" {
  vpc_id = "${aws_vpc.create_vpc.id}"
  tags {
    Name = "internet-gateway"
  }
}

resource "aws_internet_gateway" "igw_2" {
  provider = "aws.us-east-2"
  vpc_id = "${aws_vpc.create_vpc_2.id}"
  tags {
    Name = "internet-gateway"
  }
}

~~~

외부 접속을 위해 internet gateway도 생성해준다.

~~~terraform

# vpc끼리 통신하기 위해서 라우팅 테이블 수정
resource "aws_route" "internet_access_vpc" {
    route_table_id = "${aws_vpc.create_vpc.main_route_table_id}"
    destination_cidr_block = "${aws_vpc.create_vpc_2.cidr_block}"
    vpc_peering_connection_id = "${aws_vpc_peering_connection.default.id}"
}

## 외부에서 ssh 접속하기 위해서 internet gateway 연결
resource "aws_route" "internet_access" {
    route_table_id = "${aws_vpc.create_vpc.main_route_table_id}"
    destination_cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.igw_1.id}"
}

# vpc끼리 통신하기 위해서 라우팅 테이블 수정
resource "aws_route" "internet_access_vpc_2" {
    provider = "aws.us-east-2"
    route_table_id = "${aws_vpc.create_vpc_2.main_route_table_id}"
    destination_cidr_block = "${aws_vpc.vpc_test.cidr_block}"
    vpc_peering_connection_id = "${aws_vpc_peering_connection.default.id}"
}

## 외부에서 ssh 접속하기 위해서 internet gateway 연결
resource "aws_route" "internet_access_2" {
    provider = "aws.us-east-2"
    route_table_id = "${aws_vpc.create_vpc_2.main_route_table_id}"
    destination_cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.igw_2.id}"
}

~~~

라우팅 테이블에 vpc의 cidr_block을 등록 해주고, 외부 접속을 위해 0.0.0.0/0 도 등록해준다.

마지막으로 vpc peering을 하자.

~~~terraform

resource "aws_vpc_peering_connection" "default" {
  peer_owner_id = "871229912567"
  peer_vpc_id   = "${aws_vpc.create_vpc_2.id}"
  vpc_id        = "${aws_vpc.vpc_test.id}"
  peer_region   = "us-east-2"
  auto_accept   = false

  tags = {
    Side = "Requester"
  }
}
resource "aws_vpc_peering_connection_accepter" "default" {
  provider                  = "aws.us-east-2"
  vpc_peering_connection_id = "${aws_vpc_peering_connection.default.id}"
  auto_accept               = true

  tags = {
    Side = "Accepter"
  }
}

~~~

peer_vpc_id 에 연결할 region의 vpc를 등록해주고 vpc_id 에 현재 region의 vpc를 등록해준다. 

resource에 provider를 명시 안해줬으므로 default region은 us-east-1로 잡혀 있으므로 peer_region에는 연결 시킬 us-east-2를 입력 해주었다.

`전체 코드`
 [https://github.com/jaejin1/VPC-Peering/blob/master/main.tf](https://github.com/jaejin1/VPC-Peering/blob/master/main.tf) 

