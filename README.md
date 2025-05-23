Вітаю!
спочатку ip на строінку main таска http://3.110.103.112
в цьому файлі я навів виконання extra таску, VPC (Різні VPC: Повторіть кроки 2–5 для інстансів у різних VPC з внутрішнім і зовнішнім зв’язком.)

Я виконував через cli оскільки якщо використовувати ui воно буде майже автоматично тож: 
створення 1 vpc з pub sub.
~ $ aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=PublicVPC}]'
~ $ aws ec2 create-subnet \
>   --vpc-id vpc-019f630f76a251651 \
>   --cidr-block 10.0.1.0/24 \
>   --availability-zone ap-south-1a \
>   --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=MyPublicSubnet}]'
> 
~ $ aws ec2 create-subnet \
>   --vpc-id vpc-019f630f76a251651 \
>   --cidr-block 10.0.1.0/24 \
>   --availability-zone ap-south-1a \
>   --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=MyPublicSubnet}]'

{
    "Subnet": {
        "AvailabilityZoneId": "aps1-az1",
        "OwnerId": "354918390786",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {

~ $ aws ec2 create-internet-gateway \
>   --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'

{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-02cde498fe34bef89",
        "OwnerId": "354918390786",
        "Tags": [
            {
                "Key": "Name",
                "Value": "MyIGW"
            }
        ]
    }
}
~ $ 
~ $ aws ec2 attach-internet-gateway \
>   --internet-gateway-id igw-02cde498fe34bef89 \
>   --vpc-id vpc-019f630f76a251651 \
>   --region ap-south-1

~ $ 
~ $ aws ec2 create-route-table \
>   --vpc-id vpc-019f630f76a251651 \
>   --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]' \
>   --region ap-south-1

{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "Routes": [
            {
                "DestinationCidrBlock": "10.0.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [
            {
                "Key": "Name",
                "Value": "PublicRouteTable"
            }
        ],
        "VpcId": "vpc-019f630f76a251651",
        "OwnerId": "354918390786"

~ $ aws ec2 create-route   --route-table-id rtb-068137cdebdd14455   --destination-cidr-block 0.0.0.0/0   --gateway-id igw-02cde498fe34bef89   --region ap-south-1                                                             
{
    "Return": true
}
~ $ aws ec2 describe-subnets \
>   --filters "Name=vpc-id,Values=vpc-019f630f76a251651" \
>   --region ap-south-1 \
>   --query "Subnets[*].{ID:SubnetId, CIDR:CidrBlock, Name:Tags[?Key=='Name']|[0].Value}" \
>   --output table
---------------------------------------------------------------
|                       DescribeSubnets                       |
+-------------+----------------------------+------------------+
|    CIDR     |            ID              |      Name        |
+-------------+----------------------------+------------------+
|  10.0.1.0/24|  subnet-0bc820d5e0d0da216  |  MyPublicSubnet  |
+-------------+----------------------------+------------------+

~ $ aws ec2 associate-route-table   --subnet-id subnet-0bc820d5e0d0da216   --route-table-id rtb-068137cdebdd14455   --region ap-south-1
{
    "AssociationId": "rtbassoc-080057469626addd9",
    "AssociationState": {
        "State": "associated"
    }
}
~ $ aws ec2 modify-subnet-attribute   --subnet-id subnet-0bc820d5e0d0da216   --map-public-ip-on-launch   --region ap-south-1


створюю private VPC
~ $ aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=PrivateVPC}]' \
  --region ap-south-1
~ $ aws ec2 create-subnet   --vpc-id vpc-08b706e7a8c2d90ba  --cidr-block 10.1.1.0/24   --availability-zone ap-south-1a   --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PrivateSubnet}]'   --region ap-south-1
~ $ aws ec2 allocate-address --domain vpc --region ap-south-1
{
    "AllocationId": "eipalloc-0eae75a6bf9d512e8",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "ap-south-1",
    "Domain": "vpc",
    "PublicIp": "13.234.210.131"
}

~ $ 
~ $ aws ec2 create-nat-gateway  --subnet-id subnet-0bc820d5e0d0da216  --allocation-id eipalloc-0eae75a6bf9d512e8  --region ap-south-1
{
    "ClientToken": "ed47569b-0826-4bfa-addc-2a31661ee835",
    "NatGateway": {
        "CreateTime": "2025-05-23T18:58:27+00:00",
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-0eae75a6bf9d512e8",
                "IsPrimary": true,

~ $ 
~ $ aws ec2 create-route-table --vpc-id vpc-08b706e7a8c2d90ba --region ap-south-1

{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-0aff7e0487af5a0d1",
        "Routes": [
            {
                "DestinationCidrBlock": "10.1.0.0/16",
                "GatewayId": "local",

~ $ 
~ $ aws ec2 create-vpc-peering-connection \
>   --vpc-id vpc-019f630f76a251651 \
>   --peer-vpc-id vpc-08b706e7a8c2d90ba \
>   --region ap-south-1 \
>   --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=PublicToPrivatePeering}]'
{
    "VpcPeeringConnection": {
        "AccepterVpcInfo": {
            "OwnerId": "354918390786",
            "VpcId": "vpc-08b706e7a8c2d90ba",
            "Region": "ap-south-1"
        },
        "ExpirationTime": "2025-05-30T19:08:53+00:00",

~ $ 
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0f07c2276de1c6a69 \
  --region ap-south-1

~ $ aws ec2 create-route \
>   --route-table-id rtb-068137cdebdd14455 \
>   --destination-cidr-block 10.1.0.0/16 \
>   --vpc-peering-connection-id pcx-0f07c2276de1c6a69 \
>   --region ap-south-1
{
    "Return": true
}
~ $
> ~ $ aws ec2 create-route \
>   --route-table-id rtb-0aff7e0487af5a0d1 \
>   --destination-cidr-block 10.0.0.0/16 \
>   --vpc-peering-connection-id pcx-0f07c2276de1c6a69 \
>   --region ap-south-1
{
    "Return": true
}
~ $
> aws ec2 create-security-group \
  --group-name PublicInstanceSG \
  --description "SG for public instance in PublicVPC" \
  --vpc-id vpc-019f630f76a251651 \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=PublicInstanceSG}]' \
  --region ap-south-1
aws ec2 authorize-security-group-ingress \
  --group-id sg-0d5ce9cdbedff8368 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0 \
  --region ap-south-1
>
~ $ aws ec2 authorize-security-group-ingress   --group-id sg-0d5ce9cdbedff8368   --protocol tcp   --port 80   --cidr 0.0.0.0/0   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-04530ba9bb23b6213",
            "GroupId": "sg-0d5ce9cdbedff8368",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
~ $ aws ec2 authorize-security-group-ingress \
>   --group-id sg-0d5ce9cdbedff8368 \
>   --protocol icmp \
>   --port -1 \
>   --cidr 0.0.0.0/0 \
>   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0f857e0310ae7c515",
            "GroupId": "sg-0d5ce9cdbedff8368",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
ДЛЯ private VPC ~ $ aws ec2 create-security-group   --group-name PrivateInstanceSG   --description "SG for private instance in PrivateVPC"   --vpc-id vpc-08b706e7a8c2d90ba   --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=PrivateInstanceSG}]'   --region ap-south-1
{
    "GroupId": "sg-065e4f809b64715c3",
    "Tags": [
        {
            "Key": "Name",
            "Value": "PrivateInstanceSG"
        }
    ],

~ $  
~ $ aws ec2 authorize-security-group-ingress   --group-id sg-0d5ce9cdbedff8368   --protocol tcp   --port 22   --cidr 0.0.0.0/0   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0178bad08631a11af",
            "GroupId": "sg-0d5ce9cdbedff8368",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
~ $ aws ec2 authorize-security-group-ingress   --group-id sg-065e4f809b64715c3   --protocol tcp   --port 22   --cidr 10.0.0.0/16   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0d035a86f177ac74d",
            "GroupId": "sg-065e4f809b64715c3",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
~ $  aws ec2 authorize-security-group-ingress   --group-id sg-065e4f809b64715c3   --protocol icmp   --port -1   --cidr 10.0.0.0/16   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0022c5c468c009e98",
            "GroupId": "sg-065e4f809b64715c3",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
~ $  aws ec2 authorize-security-group-ingress   --group-id sg-065e4f809b64715c3   --protocol tcp   --port 80   --cidr 10.0.0.0/16   --region ap-south-1
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-00e628e93e558f2b4",
            "GroupId": "sg-065e4f809b64715c3",
            "GroupOwnerId": "354918390786",
            "IsEgress": false,

~ $ 
~ $ aws ec2 associate-route-table \
>   --subnet-id subnet-0ad69eec9963b9f25 \
>   --route-table-id rtb-0aff7e0487af5a0d1 \
>   --region ap-south-1
{
    "AssociationId": "rtbassoc-033b7a12d43b10602",
    "AssociationState": {
        "State": "associated"
    }
}
~ $


(Я хотів довести до кінця але вже під кінець набридло просто робити вручну) потім щоб завершити потрібно згенерити ключ пару pem і створити 2 вмки вот посилання на доки https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/run-instances.html  і потім просто по ssh доєднатися і все буде пінгуватисб 
ДЯКУЮ!
