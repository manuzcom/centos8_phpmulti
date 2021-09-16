# CentOS 7.8 + PHP Multi
> 2021년 08월 20일 작성
> CentOS 7.8 이 올려져 있다고 가정하고 php 설치에 관한 부분만 작성

## NGINX 설정
/etc/nginx/default.d/php73.conf
버전별로 포트를 수정하여 파일을 생성해주어야 함
53 -> 80까지. 사용하는 버전만
```nginx
location ~ \.php$ {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass   127.0.0.1:9073; 
	  ## 포트를 버전별로 나누어 작성
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

/etc/nginx/conf.d/사이트별.conf
```nginx
server {
    server_name  dev01-phpinfo.site.kr;
    root /home/계정/src/;

    index index.html;

    error_log /var/log/nginx/계정-error.log error;
    access_log /var/log/nginx/계정-access.log combined;

    include default.d/security.conf;
    include default.d/default.conf;
    include default.d/phprewrite.conf;
    include default.d/php73.conf;
    ## 버전에 맞는 php 설정 로드

    listen 80;
}
```

## PHP 설치 (with php79/stack) 초기 셋팅
다양한 설치를 지원하나 환경적인 문제로 php 만 해당 툴을 이용하여 설치하도록 함.
```shell
cd ~
yum install -y git
git clone https://github.com/php79/stack.git
cd stack

# 설치할 버전선택 하여 1로 변경
vi stack.defaults.conf
# 필요한 php 버전을 1로 변경
# 나머지 모든 옵션을 0으로 변경
# INTERACTIVE = 1 로 유지 (중간에 발생하는 오류를 확인하기 위해 필요)

```

## browscap 파일 다운로드 (Full - php7이상 / Lite - php5)
```shell
# browscap 파일 다운로드 
# Full
wget http://browscap.org/stream?q=Full_PHP_BrowsCapINI -O /etc/php_browscap.ini

# Lite
wget http://browscap.org/stream?q=Lite_PHP_BrowsCapINI -O /etc/php_browscap_lite.ini

# Standard
wget http://browscap.org/stream?q=PHP_BrowsCapINI -O /etc/php_browscap.ini

# php-fpm 서비스 시작
systemctl start php-fpm
```

## stack 파일 수정 적용
/root/stack/php/53/z-php79.ini
/root/stack/php/70/z-php79.ini
/root/stack/php/80/z-php79.ini
```ini
# 파일 안에서 아래 부분 수정 및 추가
upload_max_filesize = 200M
max_file_uploads = 30
post_max_size = 300M
browscap = /etc/php_browscap.ini

# php 5는 get_browser 함수가 매우 느린 버그가 있으므로 browscap 을 lite 로 연결해주어야 함. 
# 7버전 이상은 Full 버전을 연결해도 무방함. 
browscap = /etc/php_browscap_lite.ini 
```

## browscap 파일 다운로드 (Full 추천)
```shell
# browscap 파일 다운로드 
# Full
wget http://browscap.org/stream?q=Full_PHP_BrowsCapINI -O /etc/php_browscap.ini

# Lite
wget http://browscap.org/stream?q=Lite_PHP_BrowsCapINI -O /etc/php_browscap.ini

# Standard
wget http://browscap.org/stream?q=PHP_BrowsCapINI -O /etc/php_browscap.ini
```

## stack 이어서 진행
```shell
cd /root/stack
./install.sh
Y 눌러서 진행.

## 혹시 에러가 나서 중단된 부분이 있다면
# locks 폴더 안의 중단되어 설치 되지 않은 스크립트를 삭제해야함.

# 설정파일을 한곳에 모아 관리 편하게 셋팅
# 필요한 버전만 셋팅
mkdir /etc/php.d

ln -s /usr/local/php53/etc/php.d/z-php79.ini /etc/php.d/php53.ini
ln -s /opt/remi/php54/root/etc/php.d/z-php79.ini /etc/php.d/php54.ini
ln -s /opt/remi/php55/root/etc/php.d/z-php79.ini /etc/php.d/php55.ini
ln -s /opt/remi/php56/root/etc/php.d/z-php79.ini /etc/php.d/php56.ini
ln -s /etc/opt/remi/php70/php.d/z-php79.ini /etc/php.d/php70.ini
ln -s /etc/opt/remi/php71/php.d/z-php79.ini /etc/php.d/php71.ini
ln -s /etc/opt/remi/php72/php.d/z-php79.ini /etc/php.d/php72.ini
ln -s /etc/opt/remi/php73/php.d/z-php79.ini /etc/php.d/php73.ini
ln -s /etc/opt/remi/php74/php.d/z-php79.ini /etc/php.d/php74.ini
ln -s /etc/opt/remi/php80/php.d/z-php79.ini /etc/php.d/php80.ini

## php-fpm 로그파일 /var/log 밑에 모으기
mkdir /var/log/php-fpm/

ln -s /usr/local/php53/var/log/php-fpm.log /var/log/php-fpm/php53-fpm.log
ln -s /opt/remi/php54/root/var/log/php-fpm/www-error.log /var/log/php-fpm/php54-fpm.log
ln -s /opt/remi/php55/root/var/log/php-fpm/www-error.log /var/log/php-fpm/php55-fpm.log
ln -s /opt/remi/php56/root/var/log/php-fpm/www-error.log /var/log/php-fpm/php56-fpm.log
ln -s /var/opt/remi/php70/log/php-fpm/www-error.log /var/log/php-fpm/php70-fpm.log
ln -s /var/opt/remi/php71/log/php-fpm/www-error.log /var/log/php-fpm/php71-fpm.log
ln -s /var/opt/remi/php72/log/php-fpm/www-error.log /var/log/php-fpm/php72-fpm.log
ln -s /var/opt/remi/php73/log/php-fpm/www-error.log /var/log/php-fpm/php73-fpm.log
ln -s /var/opt/remi/php74/log/php-fpm/www-error.log /var/log/php-fpm/php74-fpm.log
ln -s /var/opt/remi/php80/log/php-fpm/www-error.log /var/log/php-fpm/php80-fpm.log

## php-fpm 설정 파일 모으기
mkdir /etc/php-fpm.d/

ln -s /usr/local/php53/etc/php-fpm.conf /etc/php-fpm.d/php53-fpm.conf
ln -s /opt/remi/php54/root/etc/php-fpm.d/www.conf /etc/php-fpm.d/php54-fpm.conf
ln -s /opt/remi/php55/root/etc/php-fpm.d/www.conf /etc/php-fpm.d/php55-fpm.conf
ln -s /opt/remi/php56/root/etc/php-fpm.d/www.conf /etc/php-fpm.d/php56-fpm.conf
ln -s /etc/opt/remi/php70/php-fpm.d/www.conf /etc/php-fpm.d/php70-fpm.conf
ln -s /etc/opt/remi/php71/php-fpm.d/www.conf /etc/php-fpm.d/php71-fpm.conf
ln -s /etc/opt/remi/php72/php-fpm.d/www.conf /etc/php-fpm.d/php72-fpm.conf
ln -s /etc/opt/remi/php73/php-fpm.d/www.conf /etc/php-fpm.d/php73-fpm.conf
ln -s /etc/opt/remi/php74/php-fpm.d/www.conf /etc/php-fpm.d/php74-fpm.conf
ln -s /etc/opt/remi/php80/php-fpm.d/www.conf /etc/php-fpm.d/php80-fpm.conf

## php-fpm 재시작 필요시
systemctl restart php53-php-fpm
systemctl restart php54-php-fpm
systemctl restart php55-php-fpm
systemctl restart php56-php-fpm
systemctl restart php70-php-fpm
systemctl restart php71-php-fpm
systemctl restart php72-php-fpm
systemctl restart php73-php-fpm
systemctl restart php74-php-fpm
systemctl restart php80-php-fpm

```
