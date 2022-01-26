## Задание

## 1. Создайте виртуальную машину Linux.  ![1](https://user-images.githubusercontent.com/33546071/150525487-87c567f3-8e1b-4e6f-89c2-ded7c2631f68.jpg)

## 2. Установите ufw и разрешите к этой машине сессии на порты 22 и 443, при этом трафик на интерфейсе localhost (lo) должен ходить свободно на все порты.  

apt install ufw
ufw allow 443
ufw allow 80
ufw enable
ufw status numbered  
Status: active  
  
   To                         Action      From  
     --                         ------      ----  
[ 1] 22                         ALLOW IN    Anywhere  
[ 2] 443                        ALLOW IN    Anywhere  
[ 3] 22 (v6)                    ALLOW IN    Anywhere (v6)  
[ 4] 443 (v6)                   ALLOW IN    Anywhere (v6)  


## 3. Установите hashicorp vault ([инструкция по ссылке](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started#install-vault)).  

Установка достаточно простая:  

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -  
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"  
sudo apt-get update && sudo apt-get install vault  

## 4. Cоздайте центр сертификации по инструкции ([ссылка](https://learn.hashicorp.com/tutorials/vault/pki-engine?in=vault/secrets-management)) и выпустите сертификат для использования его в настройке веб-сервера nginx (срок жизни сертификата - месяц).

apt-get install jq  
Открываем 2 терминала, в первом: 

vault server -dev -dev-root-token-id root  

во втором:  

export VAULT_ADDR=http://127.0.0.1:8200  
export VAULT_TOKEN=root  

Генерируем root CA сертификат:  
vault secrets enable pki  
vault secrets tune -max-lease-ttl=87600h pki  
vault write -field=certificate pki/root/generate/internal \  
     common_name="example.com" \  
     ttl=87600h > CA_cert.crt  
vault write pki/config/urls \  
     issuing_certificates="$VAULT_ADDR/v1/pki/ca" \  
     crl_distribution_points="$VAULT_ADDR/v1/pki/crl"  
     
Генерируем промежуточный сертификат:  
vault secrets enable -path=pki_int pki  
vault secrets tune -max-lease-ttl=43800h pki_int  
vault write -format=json pki_int/intermediate/generate/internal \  
     common_name="example.com Intermediate Authority" \  
     | jq -r '.data.csr' > pki_intermediate.csr  
vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr \  
     format=pem_bundle ttl="43800h" \  
     | jq -r '.data.certificate' > intermediate.cert.pem  
     
Создаем роль с именем example-dot-com  

vault write pki_int/roles/example-dot-com \  
     allowed_domains="example.com" \  
     allow_subdomains=true \ 
     max_ttl="720h"  

Выпустим сертификаты  

vault write pki_int/issue/example-dot-com common_name="test.example.com" ttl="30d"  

В выводе команды будет закрытый ключ, сертификат. Так же сертификат можно выпустить в web - инетрфейсе. Сертификаты (начинаются с -----BEGIN CERTIFICATE-----  

и заканивающиеся на -----END CERTIFICATE-----) сохраняем с помощью блокнота с расширение .cer. Не зыбваем сохранить закрытый ключ.


##5. Установите корневой сертификат созданного центра сертификации в доверенные в хостовой системе.  

Так выглядит цепочка сертификатов после установки в систему корневого и промежуточного сертификата  

![image](https://user-images.githubusercontent.com/33546071/151161276-141d3129-2a92-4d35-b822-a267c10cbe23.png)  

## 6. Установите nginx.

apt install nginx  

После установки nginx по адресу http://IP адрес виртуальной машины/ стала доступна стандартная стартовая страница  

## 7. По инструкции ([ссылка](https://nginx.org/en/docs/http/configuring_https_servers.html)) настройте nginx на https, используя ранее подготовленный сертификат:  
  - можно использовать стандартную стартовую страницу nginx для демонстрации работы сервера;  
  - можно использовать и другой html файл, сделанный вами;  

Вносим изменения в конфигурацию сайта:  

nano /etc/nginx/sites-available/default  

Приводим файл к примерно такому виду:  

server {
        listen 80 default_server;  
        listen [::]:80 default_server; 
        listen              443 ssl;  
        server_name         test.example.com;  
        ssl_certificate     /etc/nginx/ssl/cert.crt;  
        ssl_certificate_key /etc/nginx/ssl/key.key;  
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;  
        ssl_ciphers         HIGH:!aNULL:!MD5;  

Создаем директорию /etc/nginx/ssl и копируем туда файлы сертификата и ключа, как указано в файле конфигурации  

systemctl restart nginx (перезапускаем nginx для применения настроек)  


## 8. Откройте в браузере на хосте https адрес страницы, которую обслуживает сервер nginx.

Для корректного отобращения сайта я добавил в файл хост соотношение сайта (test.example.com) и ip адреса виртуальной машины на которой nginx  
После чего стал корректно отображаться сайт https://test.example.com, браузер уведомил что "соединение безопасно" и сертификат валидный  
![image](https://user-images.githubusercontent.com/33546071/151166706-8456fb63-2517-4d37-8366-29ccd135565b.png)


## 9. Создайте скрипт, который будет генерировать новый сертификат в vault:
  - генерируем новый сертификат так, чтобы не переписывать конфиг nginx;
  - перезапускаем nginx для применения нового сертификата.
## 10. Поместите скрипт в crontab, чтобы сертификат обновлялся какого-то числа каждого месяца в удобное для вас время.



## Результат

Результатом курсовой работы должны быть снимки экрана или текст:

- Процесс установки и настройки ufw
- Процесс установки и выпуска сертификата с помощью hashicorp vault
- Процесс установки и настройки сервера nginx
- Страница сервера nginx в браузере хоста не содержит предупреждений 
- Скрипт генерации нового сертификата работает (сертификат сервера ngnix должен быть "зеленым")
- Crontab работает (выберите число и время так, чтобы показать что crontab запускается и делает что надо)
