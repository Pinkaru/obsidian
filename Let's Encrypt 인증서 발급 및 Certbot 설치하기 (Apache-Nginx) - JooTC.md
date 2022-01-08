author:: ()x
source:: [Let's Encrypt 인증서 발급 및 Certbot 설치하기 (Apache/Nginx) - JooTC](https://jootc.com/p/201901062488)
clipped:: [[2022-01-06]]
published:: 

#clippings

![letsencrypt_and_certbot_logo](https://cdn.jootc.com/wp-content/uploads/2019/01/letsencrypt_and_certbot_logo.png)

## 웹사이트 인증서, 그리고 Let’s Encrypt란?

**SSL 또는 TLS 인증서**는 웹 사이트를 운영하면서 거의 필수적인 요소입니다. 클라이언트와 서버 간의 통신이 발생할 때 전송되는 모든 패킷 데이터를 암호화하여 감청이나 식별을 어렵게하여 보안에 있어 강력한 역할을 합니다.

웹 사이트 인증서를 발급해주는 대표적인 **발급 기관(CA)**으로는 **[Verisign](https://www.verisign.com/ "Verisign")**, [**Comodo**](https://www.comodo.com/ "Comodo")나 [**GlobalSign**](https://www.globalsign.com/en/ "GlobalSign") 등이 있으며 대개 호스팅 업체에서 등록을 대행해주기도 합니다.

민감한 정보를 교환하거나 개인정보를 취급하는 웹 사이트는 **SSL/TLS 인증서를 발급**하는 것이 좋습니다.

특히 법률에 의해 개인정보를 취급하는 웹 사이트에는 SSL 인증서를 무조건 달아야 할 것입니다. 그러나 대부분의 인증서 발급은 **무료가 아니기 때문에** 매년 새로운 인증서를 갱신함으로서 발생하는 시간과 비용은 만만치 않습니다. 물론 무료 인증서도 몇몇 있었지만 인증서 기관의 신용이 높은 편은 아닙니다.

이러한 가격적인 부담을 줄이고 인증서 발급의 복잡한 절차를 줄이고자 등장한 것은 [**Let’s Encrypt**](https://letsencrypt.org/ "Let’s Encrypt")라는 비영리 기관입니다.

2016년 4월이라는 오래되지 않은 시기에 등장하였지만 현재는 **모질라(Mozilla)**와 **시스코(Cisco)**, **구글(Google)** 등의 다양한 업체가 스폰서로 참여하고 있으며 수많은 웹 사이트에 널리 사용되고 있는 추세입니다.

### **Let’s Encrypt**를 사용하면 어떤 점이 좋은가요?

-   **인증 절차가 단순화**되었을 뿐만 아니라 **발급 대기 시간이 없어** 빠르게 인증서를 적용할 수 있습니다.
-   **TLS 인증서** 발급이 가능하며 **와일드 카드 인증서**를 지원합니다.
-   발급을 위한 정보는 발급자 **이메일**만 요구됩니다.
-   만료된 인증서 갱신을 **자동화**할 수 있습니다.
-   심지어 **무료**입니다!

## 웹사이트 인증서를 발급하기 전에

먼저 인증서 발급을 위해서는 아래와 같은 **조건**이 필요합니다.

-   인증서를 설치할 서버의 **터미널**에 접속할 수 있어야 하며 **root 권한**을 사용할 수 있어야 합니다.

만약 일부 기능이 제한된 **웹 호스팅 서비스**를 사용 중이라면, 해당 업체에 문의하여 **사용/설치 가능 여부를 확인**해보셔야 합니다.

예를 들어 **카페24**나 **가비아**에서 서비스하는 웹 서비스 호스팅(서버 호스팅과는 다름)의 경우 유료 SSL 구매 및 설치만 가능하므로 불가능할 수 있습니다. 그러나 가상 또는 물리 서버를 **구매/임대**하였거나 **클라우드 서비스(AWS, Azure 등)**를 사용 중인 경우 서버 내 터미널에 접속할 수 있다면 Let’s Encrypt 인증서를 설치할 수 있습니다.

또한 터미널에 접속하여 **root 권한**(또는 root 계정 로그인)을 사용할 수 있어야 합니다. 일부 호스팅 업체에선 root 권한 취득을 제한하기도 합니다. 대표적으로 SSH 프로토콜을 사용하여 원격으로 서버에 접속할 수 있으며 해당 내용은 **여기([https://jootc.com/p/201808031462](https://jootc.com/p/201808031462 "https://jootc.com/p/201808031462"))**에 설명되어 있습니다.

**Let’s Encrypt**는 **퍼블릭** **도메인이 할당된 서버**에서만 발급이 가능합니다. 내부 테스트용으로 구성된 서버이거나 공개 서버가 아닌 등의 이유로 자신의 서버에 IP만 할당되어 있는 경우에는 인증서 발급과 설치에 어려움이 있으므로 반드시 도메인을 할당해주어야 합니다.

**인증서 유효기간은 90일**이므로 지속적인 갱신 과정이 필요합니다. 다행히 갱신 과정을 자동화할 수 있음은 물론 Let’s Encrypt에서도 자동화에 대한 규제를 하지는 않기 때문에 유효기간에 있어 특별한 문제는 없을 것입니다. (물론 자동화 과정이 쉽지는 않습니다.)

그 외에 Let’s Encrypt 인증서에 대해 **주의해야 할 내용**은 다음과 같습니다.

-   인증서로 인해 발생한 피해에 대해 해당 기관으로부터 **보상 받을 수 없습니다**.
-   일부 오래 된 운영체제나 브라우저에서 인증서로 인한 올바르지 않은 동작이 발생할 수 있습니다.

## 인증서 설치 1단계 – Certbot 설치하기

**Let’s Encrypt** 인증서를 설치하기 위해서는 **Certbot**이라는 커맨드라인 도구를 사용해야 합니다. Certbot 도구는 수많은 웹 서비스와 운영체제를 지원하며 각각의 환경에 따라 설치 방법이 달라질 수 있습니다.

이 포스트에서는 가장 많이 쓰이는 리눅스 운영체제인 **CentOS/Ubuntu**, 그리고 웹 서비스 **Apache와 Nginx**를 중심으로 기술되었습니다.

설치를 위한 **테스트 환경**은 다음과 같이 구성해보았습니다.

-   운영체제 : Linux CentOS 7
-   도메인 : www.example.com (**주의 –** 이 도메인으로 실제로 테스트하시면 안됩니다.)
-   웹 서비스 : Nginx

먼저 인증서 발급에 필요한 패키지를 설치해주어야 합니다.

### \[Step 1-1\] CentOS 설치 (CentOS 7 기준)

**CentOS**의 경우 **EPEL 저장소**를 활성화시켜주어야 합니다. 다음 명령어로 **EPEL-Repository**를 설치합니다.

$ sudo yum install epel-release

$ sudo yum install epel-release

$ sudo yum install epel-release

### \[Step 1-2\] Ubuntu 설치 (18.04 LTS 기준)

**우분투(Ubuntu)**에서는 다음 명령어를 하나씩 실행시켜줍니다.

$ sudo apt-get install software-properties-common

$ sudo add-apt-repository universe

$ sudo add-apt-repository ppa:certbot/certbot

$ sudo apt-get update $ sudo apt-get install software-properties-common $ sudo add-apt-repository universe $ sudo add-apt-repository ppa:certbot/certbot $ sudo apt-get update

$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update

### \[Step 2\] Certbot 설치

상단의 운영체제별 설치가 완료되었다면 이제 Certbot 패키지를 설치해줍니다. 패키지관리자는 운영체제에 따라 서로 다르게 입력합니다. (하단에서는 yum으로 진행합니다.)

-   **RedHat 계열 / CentOS :** yum
-   **Debian 계열 / Ubuntu :** apt

이제 `certbot`을 설치해보겠습니다.

$ sudo yum install certbot

$ sudo yum install certbot

$ sudo yum install certbot

이후 각각의 웹 서비스에 맞는 플러그인을 설치해줍니다.

### Apache

$ sudo yum install python2-certbot-apache

$ sudo yum install python2-certbot-apache

$ sudo yum install python2-certbot-apache

### Nginx

$ sudo yum install python2-certbot-nginx

$ sudo yum install python2-certbot-nginx

$ sudo yum install python2-certbot-nginx

## 인증서 설치 2단계 – 발급 과정 파악하기

이제 설치 작업은 완료되었으며 본격적으로 **Certbot**을 사용하여 인증서를 생성해보도록 하겠습니다.

발급할 때 주의해야 할 점은 하나의 호스트 또는 도메인에서 1일에 3회 이상의 발급을 시도할 수 없으므로 가능하면 발급 절차 시 실수를 하지 않도록 해야 합니다.

인증서를 발급하는 방법은 주로 **webroot와 Standalone, DNS**의 3가지 방식이 있습니다.

먼저 **webroot** 방식은 실제 웹 디렉토리 내에 인증서의 유효성을 확인할 수 있는 파일을 업로드하여 인증서를 발급하는 방법입니다. 한 번의 명령에 하나의 도메인 인증서만 발급받을 수 있습니다.

다음으로 **Standalone** 방식은 일시적으로 호스트 내의 웹 서비스를 빌려 인증서 유효성을 확인하는 방법입니다. 여러 도메인을 발급받을 수 있으나 이 방법을 사용하면 인증서가 발급되는 동안 운영 중인 **웹 서비스가 잠시 중단**될 것입니다.

마지막으로 **DNS** 방식은 도메인을 쿼리하여 나타나는 TXT 레코드에서 인증서 유효성을 확인하는 방법입니다. 이 경우 해**당 도메인의 DNS를 관리/수정할 수 있는 조건**이 되어야 합니다.

결론적으로 **인증 기관(CA)**이 발급할 서버에 방문하여 인증서와 동일한 도메인인지 확인하는 과정이 진행되며, 이러한 과정에 **ACME 프로토콜**을 사용하여 CA와 서버 간의 유효성 검증을 시도**(Challenge)** 합니다. 따라서 투명성을 위해 인증 도중 서버의 IP와 트랜잭션 내역이 인증 기관에 기록되게 됩니다.

아무튼 다양한 발급 방법들이 있지만, 여기에는 각자의 장단점이 있고 어떠한 방식으로 해도 무관하므로 원하시는 방법을 선택하여 진행하면 되겠습니다.

## 인증서 설치 3단계 – Certbot 명령어 사용

**Certbot**의 일반적인 명령 예시는 다음과 같습니다.

$ sudo certbot --apache --standalone -d example.com certonly

$ sudo certbot --apache --standalone -d example.com certonly

$ sudo certbot --apache --standalone -d example.com certonly

### –apache / –nginx

기본 명령에 **웹 서비스(예 : apache, nginx 등) 이름**을 옵션값으로 붙여주어 해당 웹 서비스에 맞는 발급 과정을 진행하도록 합니다.

### –standalone / –webroot

인증서 발급 방식은 옵션을 붙여 사용할 수 있습니다. (예 : webroot 방식인 경우 `--webroot`를 붙입니다.) 세가지 방식을 직접 선택할 경우 `--manual` 옵션을 붙이면 됩니다.

DNS 방식의 경우 `--preferred-challenges dns` 값을 붙여줍니다.

### \-d \[도메인 이름\]

\-d 옵션은 발급할 인증서의 도메인을 입력하는 부분입니다. 여러 도메인을 사용하는 경우 계속 -d 옵션을 붙이거나 콤마(,)로 구분하여 입력하면 됩니다. (예 : `-d example.com,sub1.example.com,sub2.example.com...`)

### certonly

원래는 인증서 발급 시 웹 서비스의 설정 파일을 직접 편집해줍니다. (다만 가능하다면 직접 수정하는 것이 안전합니다.) `certonly` 값을 붙이면 웹 서비스 설정 파일에 인증서 관련 내용을 임의로 수정하지 않도록 합니다.

이제 파악한 정보를 토대로 본격적인 **인증서를 발급**해보도록 하겠습니다.

여기서는 TXT 레코드를 수정하거나 임의의 파일을 생성할 필요가 없는 **Standalone** 방식으로 진행할 것입니다. 사실상 간편하고 빠르기 때문이기도 합니다.

**Standalone** 방식은 웹 서비스를 임시로 빌리기 때문에 현재 구동되고 있는 **웹 서비스를 잠시 중지**하도록 하겠습니다. 아래는 **Nginx** 웹 서비스를 중지하는 예시입니다. 만약 **Apache**를 사용한다면 `httpd` 또는 `apache2`로 시도해보시면 됩니다.

$ sudo service nginx stop

$ sudo service nginx stop

$ sudo service nginx stop

이제 다음 명령을 입력해보겠습니다.

$ sudo certbot certonly --standalone -d example.com

$ sudo certbot certonly --standalone -d example.com

$ sudo certbot certonly --standalone -d example.com

먼저 다음과 같은 내용이 나타나면 도메인 관리자의 **이메일 주소**를 입력해줍니다. 해당 이메일로 갱신 알림이나 주요한 소식들이 발송될 수 있습니다.

Saving debug log to /var/log/letsencrypt/letsencrypt.log

Plugins selected: Authenticator standalone, Installer None

Enter email address (used for urgent renewal and security notices) (Enter 'c' to

Saving debug log to /var/log/letsencrypt/letsencrypt.log Plugins selected: Authenticator standalone, Installer None Enter email address (used for urgent renewal and security notices) (Enter 'c' to cancel):

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel):

이번에는 **이용원칙 동의** 및 인증 기관에 등록되는 사항에 관한 내용입니다.

Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Please read the Terms of Service at

https://letsencrypt.org/documents/LE\-SA-v1.2\-November-15\-2017.pdf. You must

agree in order to register with the ACME server at

https://acme-v02.api.letsencrypt.org/directory

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - Please read the Terms of Service at https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must agree in order to register with the ACME server at https://acme-v02.api.letsencrypt.org/directory - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - (A)gree/(C)ancel:

Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel:

어차피 동의해야 하므로 `A`를 입력하고 엔터를 입력해줍니다.

이번 내용은 제 3자 업체에게 정보를 공유하겠냐는 내용입니다. 원치 않은 경우 `N`을 입력하여 거부할 수 있습니다.

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Would you be willing to share your email address with the Electronic Frontier

Foundation, a founding partner of the Let's Encrypt project and the non-profit

organization that develops Certbot? We'd like to send you email about our work

encrypting the web, EFF news, campaigns, and ways to support digital freedom.

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - Would you be willing to share your email address with the Electronic Frontier Foundation, a founding partner of the Let's Encrypt project and the non-profit organization that develops Certbot? We'd like to send you email about our work encrypting the web, EFF news, campaigns, and ways to support digital freedom. - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - (Y)es/(N)o:

\- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:

이제 특별한 문제가 나타나지 않는다면 발급 과정이 진행됩니다.

Saving debug log to /var/log/letsencrypt/letsencrypt.log

Plugins selected: Authenticator standalone, Installer None

Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org

Cert is due for renewal, auto-renewing...

Renewing an existing certificate

Performing the following challenges:

http-01 challenge for example.com

Waiting for verification...

Resetting dropped connection: acme-v02.api.letsencrypt.org

\- Congratulations! Your certificate and chain have been saved at:

/etc/letsencrypt/live/example.com/fullchain.pem

Your key file has been saved at:

/etc/letsencrypt/live/example.com/privkey.pem

Your cert will expire on 2019\-04\-06. To obtain a new or tweaked

version of this certificate in the future, simply run certbot

again. To non-interactively renew \*all\* of your certificates, run

\- If you like Certbot, please consider supporting our work by:

Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate

Donating to EFF: https://eff.org/donate-le

Saving debug log to /var/log/letsencrypt/letsencrypt.log Plugins selected: Authenticator standalone, Installer None Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org Cert is due for renewal, auto-renewing... Renewing an existing certificate Performing the following challenges: http-01 challenge for example.com Waiting for verification... Cleaning up challenges Resetting dropped connection: acme-v02.api.letsencrypt.org IMPORTANT NOTES: - Congratulations! Your certificate and chain have been saved at: /etc/letsencrypt/live/example.com/fullchain.pem Your key file has been saved at: /etc/letsencrypt/live/example.com/privkey.pem Your cert will expire on 2019-04-06. To obtain a new or tweaked version of this certificate in the future, simply run certbot again. To non-interactively renew \*all\* of your certificates, run "certbot renew" - If you like Certbot, please consider supporting our work by: Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate Donating to EFF: https://eff.org/donate-le

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Starting new HTTPS connection (1): acme-v02.api.letsencrypt.org
Cert is due for renewal, auto-renewing...
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for example.com
Waiting for verification...
Cleaning up challenges
Resetting dropped connection: acme-v02.api.letsencrypt.org

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2019-04-06. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew \*all\* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
﻿

**Congratulations!** 문구가 나타났다면 발급이 정상적으로 완료된 것입니다.

## 인증서 설치 4단계 – 웹 서비스에 반영하기

인증서가 생성되면 인증서 파일은 다음 **경로**에 저장 될 것입니다.

/etc/letsencrypt/live/\[신청한 인증서의 도메인명\].com

여기서는 `example.com` 으로 시도했으므로 `/etc/letsencrypt/live/example.com` 디렉토리가 될 것입니다. (서브 도메인이어도 마찬가지입니다.)

대략적인 파일들은 다음과 같습니다.

\[root@localhost ~\]# ls -al

drwxr-xr-x. 2 root root 93 Jan 5 20:56 .

drwx------. 7 root root 136 Jan 5 21:04 ..

lrwxrwxrwx. 1 root root 34 Jan 5 20:56 cert.pem -\> ../../archive/example.com/cert1.pem

lrwxrwxrwx. 1 root root 35 Jan 5 20:56 chain.pem -\> ../../archive/example.com/chain1.pem

lrwxrwxrwx. 1 root root 39 Jan 5 20:56 fullchain.pem -\> ../../archive/example.com/fullchain1.pem

lrwxrwxrwx. 1 root root 37 Jan 5 20:56 privkey.pem -\> ../../archive/example.com/privkey1.pem

\-rw-r--r--. 1 root root 692 Jan 5 20:56 README

\[root@localhost ~\]# ls -al total 4 drwxr-xr-x. 2 root root 93 Jan 5 20:56 . drwx------. 7 root root 136 Jan 5 21:04 .. lrwxrwxrwx. 1 root root 34 Jan 5 20:56 cert.pem -> ../../archive/example.com/cert1.pem lrwxrwxrwx. 1 root root 35 Jan 5 20:56 chain.pem -> ../../archive/example.com/chain1.pem lrwxrwxrwx. 1 root root 39 Jan 5 20:56 fullchain.pem -> ../../archive/example.com/fullchain1.pem lrwxrwxrwx. 1 root root 37 Jan 5 20:56 privkey.pem -> ../../archive/example.com/privkey1.pem -rw-r--r--. 1 root root 692 Jan 5 20:56 README

\[root@localhost ~\]# ls -al
total 4
drwxr-xr-x. 2 root root  93 Jan  5 20:56 .
drwx------. 7 root root 136 Jan  5 21:04 ..
lrwxrwxrwx. 1 root root  34 Jan  5 20:56 cert.pem -> ../../archive/example.com/cert1.pem
lrwxrwxrwx. 1 root root  35 Jan  5 20:56 chain.pem -> ../../archive/example.com/chain1.pem
lrwxrwxrwx. 1 root root  39 Jan  5 20:56 fullchain.pem -> ../../archive/example.com/fullchain1.pem
lrwxrwxrwx. 1 root root  37 Jan  5 20:56 privkey.pem -> ../../archive/example.com/privkey1.pem
-rw-r--r--. 1 root root 692 Jan  5 20:56 README

(가능하면 이 파일들은 현재 위치에 그대로 두는 것이 좋습니다.)

이제 **웹 서비스의 설정 파일에 발급한 인증서를 등록**해보겠습니다. 웹 서비스의 설정 파일 특성과 환경에 따라 여기서 언급된 내용과 다르게 설정해야할 수 있으므로 아래 예시는 언제까지나 **참고용**으로만 봐주셨으면 합니다.

### Apache 설정 예시

`example.com`의 Apache 설정 파일은 일반적으로 다음 경로에 있습니다.

-   Debian 계열 : /etc/apache2/apache2.conf
-   RedHat 계열 : /etc/httpd/conf/httpd.conf

해당 경로의 파일을 편집하여 하단에 다음과 같이 `VirtualHost`를 추가해줍니다.

DocumentRoot /var/www/html

ServerName example.com:443

<VirtualHost \*:443> DocumentRoot /var/www/html ServerName example.com:443 </VirtualHost>

<VirtualHost \*:443>
DocumentRoot /var/www/html
ServerName example.com:443
</VirtualHost>

필요한 설정을 작성해주신 후 하단의 내용을 `VirtualHost` 내에 붙여넣어줍니다. (**인증서 경로**가 올바른지 다시 한 번 검토해보시기 바랍니다.)

SSLCertificateFile /etc/letsencrypt/live/example.com/cert.pem

SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem

SSLCertificateChainFile /etc/letsencrypt/live/example.com/fullchain.pem

SSLEngine on SSLCertificateFile /etc/letsencrypt/live/example.com/cert.pem SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem SSLCertificateChainFile /etc/letsencrypt/live/example.com/fullchain.pem

SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/example.com/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/example.com/fullchain.pem

이제 **중지했었던 웹 서비스를 다시 시작**해보도록 하겠습니다. 다음 명령어를 실행합니다. (Debian 계열은 `httpd` 대신 `apache2`로 입력해야 할 수 있음)

$ sudo service httpd start

$ sudo service httpd start

$ sudo service httpd start

### Nginx 설정 예시

`example.com`의 Nginx 설정 파일(`/etc/nginx/nginx.conf`) 내용은 다음과 같습니다.

listen \[::\]:443 ssl http2;

server\_name example.com www.example.com;

root /usr/share/nginx/html;

include /etc/nginx/example.com.conf.d/\*.conf;

server { listen 443 ssl http2; listen \[::\]:443 ssl http2; server\_name example.com www.example.com; root /usr/share/nginx/html; include /etc/nginx/example.com.conf.d/\*.conf; }

server {
        listen  443 ssl http2;
        listen  \[::\]:443 ssl http2;
        server\_name example.com www.example.com;
        root    /usr/share/nginx/html;
        include /etc/nginx/example.com.conf.d/\*.conf;
}

**443 포트로 listen**하고 있는 `server` 블록 내에 하단의 내용을 붙여넣습니다. (**인증서 경로**가 올바른지 다시 한 번 검토해보시기 바랍니다.)

ssl\_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot

ssl\_certificate\_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot

include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

ssl\_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

ssl\_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot ssl\_certificate\_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot ssl\_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

ssl\_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
ssl\_certificate\_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
ssl\_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

이제 **중지했었던 웹 서비스를 다시 시작**해보도록 하겠습니다. 다음 명령어를 실행합니다.

$ sudo service nginx start

$ sudo service nginx start

$ sudo service nginx start

## 인증서 설치 5단계 – 적용된 인증서 확인하기

이제 웹 사이트에 정상적으로 인증서가 설치되었는지 확인해보도록 하겠습니다. 간단히 해당 웹 사이트에 접속하여 인증서 적용 여부를 확인해보시면 됩니다.

예를 들어 대부분의 브라우저의 경우 (하단의 예시는 구글 크롬) 다음과 같이 **자물쇠 아이콘**이 나타날 것입니다.

![configured-website-ssl](https://cdn.jootc.com/wp-content/uploads/2019/01/configured-website-ssl.png)

구글 크롬의 경우 하단의 **‘인증서’** 버튼을 클릭하면 자세한 인증서 정보가 나타날 것입니다.

![configured-website-ssl-2](https://cdn.jootc.com/wp-content/uploads/2019/01/configured-website-ssl-2.png)

여기에서 발급자가 **Let’s Encrypt Authority X3**인지, 유효 기간이 올바른지 확인해보시면 됩니다.

인증서는 **90일** 동안만 유효하므로 서버 내에서 갱신을 자동화 할 수 있도록 설계하는 것이 좋습니다. 이 내용은 추가 포스팅으로 작성해보도록 하겠습니다.

## 기타 – 인증서 설치 시 다른 웹 서비스를 사용 중인 경우

이외의 웹 서비스나 운영체제를 사용 중인 경우 하단의 **Certbot 공식 웹 페이지**에서 설치 문서를 확인할 수 있습니다.

[https://certbot.eff.org/](https://certbot.eff.org/ "https://certbot.eff.org/")

![certbot-website](https://cdn.jootc.com/wp-content/uploads/2019/01/certbot-website.png)

## 참고자료

-   **Let’s Encrypt Wikipedia :** [https://ko.wikipedia.org/wiki/Let%27s\_Encrypt](https://ko.wikipedia.org/wiki/Let%27s_Encrypt "https://ko.wikipedia.org/wiki/Let%27s_Encrypt")