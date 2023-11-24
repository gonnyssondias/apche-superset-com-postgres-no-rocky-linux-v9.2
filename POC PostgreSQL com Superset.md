# Instalação do PostgreSql no Rocky Linux 9.2

## INSTALAÇÃO DO POSTEGRESQL

**INSTALANDO FERRAMENTAS**
```bash
sudo yum install rsync nfs-utils
```
**CONFIGURANDO IDIOMA DO SISTEMA OPERACIONAL**
```bash
sudo dnf install langpacks-en glibc-all-langpacks -y
```
```bash
localectl status
sudo localectl set-locale LANG=pt_BR.UTF-8
```
**INSTALANDO O POSTGRESQL 15**
```bash
cd /usr/local/src
sudo dnf install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y
sudo dnf update -y
sudo dnf install postgresql15-server -y
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
sudo systemctl start postgresql-15
sudo systemctl enable postgresql-15
sudo systemctl status postgresql-15
```
**INSTALANDO PACOTES ADICIONAIS DO POSTGRES E REQUISITOS**
```bash
sudo dnf config-manager --set-enabled crb
sudo dnf install perl-IPC-Run
sudo dnf install postgresql15-contrib
sudo dnf install postgresql15-devel
```
**CONFIGURANDO LIBS E PATH**
```bash
sudo yum install nano
sudo nano /etc/profile.d/postgresql.sh
```
***Copie e cole as variáveis abaixo, dentro do "postgresql.sh" e salve.***
```bash
export PATH=/usr/pgsql-15/bin:$PATH
export LD_LIBRARY_PATH=/usr/pgsql-15/lib:$LD_LIBRARY_PATH
export PGDATA=/dados/pgdata
```
**DÊ PERMISSÃO E EXECUTE-O.**
```bash
sudo chmod 755 /etc/profile.d/postgresql.sh
source /etc/profile.d/postgresql.sh
```
## CONFIGURANDO DATABASE E USUÁRIO NO POSTGRES

**CRIANDO USUÁRIO NO POSTGRES**
```bash
sudo -i -u postgres
```
**CHECANDO SE O USUÁRIO POSTEGRES FOI CRIADO.**
```bash
psql
```
***Saia do Usúario e depois do Postgres
Digite '\q' para sair do psql***
```bash
\q
```
***Digite 'exit' para sair do postgres***
```bash
exit
```
**ATRIBUINDO UMA SENHA AO USUÁRIO POSTEGRES**
```bash
sudo passwd postgres
```
## CONFIGURANDO O USUÁRIO SUPERSET NO POSTGRES
***Acessando o Postgres***
```bash
su - postgres
```
***ou***
```bash
sudo -i -u postgres
```

**CRIANDO UM USUÁRIO CHAMADO 'SUPERSET' QUE ACESSARÁ O BANCO DE DADOS DO SUPERSET** 
```bash
createuser --pwprompt --interactive
```
***Digite '\q' para sair do postgres***
```bash
exit
```
**CRIANDO UM BANCO DE DADOS CHAMADO SUPERSET**
```bash
createdb superset
```
**ENTRE NO DATABASE COM USUÁRIO SUPERSET** 
```bash
psql superset
```
**CRIANDO SCHEMA NO DATABASE SUPERSET** 
```bash
CREATE SCHEMA superset
```
**CONCEDENDO PRIVILÉGIOS DE ACESSO AO USUÁRIO SUPERSET**
```bash
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA superset TO superset;
```
***Digite '\q' para sair do psql***
```bash
\q
```
**CONFIGURE O ARQUIVO pg_hba.conf**

 ***Entre no arquivo pg_hba.conf e altere  ADDRESS  e  METHOD,  das linha 85 e 87, conforme descrito abaixo.***
```bash
sudo nano /var/lib/pgsql/15/data/pg_hba.conf
```
```
"local" is for Unix domain socket connections only 
local      all          all                            md5
IPv4 local connections:
host       all          all        0.0.0.0/0           md5
```
***Na coluna  METHOD na linha 85, substitua  por  md5.
Na coluna  ADDRESS na linha 87, substitua  por 0.0.0.0/0 ,  e na coluna METHOD por md5 da mesma linha***

***Entre no arquivo postgresql.conf e altere a linha 60 e 64, conforme descrito abaixo.***
```bash
sudo nano /var/lib/pgsql/15/data/postgresql.conf
```
***Retire o Hash -> '#" antes do INICIO da linha 60 e 64. E substitua por  ' * ' o que estiver dentro das aspas simples da linha 60, conforme abaixo.***
```
listen_addresses = '*'   #What IP address(es) to listen on;
.
.
port = 5432                             # (change requires restart)
```
***P.S.: Salvar e sair do arquivo, caso dê erro de permissão, usar comando abaixo e refazer procedimento acima.***
```bash
sudo su
```
**CONFIGURE O ARQUIVO config.py**

***Entre no arquivo config.py e altere as linhas 189 e 191, conforme descrito abaixo.**
```bash
sudo nano /home/<Seu usuario>/.pyenv/versions/superset/lib/python3.9/site-packages/superset/config.py
```
***Comente com Hash # a linha 189***
```
#SQLALCHEMY_DATABASE_URI = "sqlite:///" + os.path.join(DATA_DIR, "superset.db")
```
***Descomente (apagando o Hash '#') da Linha 191 e altere conforme abaixo.***
```bash
SQLALCHEMY_DATABASE_URI = postgresql+psycopg2://superset:<senha>@<host ou IP>/superset'
```
**REINICIAR O POSTGRES**
```bash
sudo systemctl restart postgresql-15
```


## INSTALE O APACHE SUPERSET

https://github.com/gonnyssondias/apache-superset_rocky_linux_v9.2.git

**APOÓS INSTALAÇÃO DO SUPERSET , CONTINUAR COM PROCEDIMENTOS ABAIXO.**
## REFAZER OS PROCEDIMENTOS NO NOVO BANCO DE DADOS.



**Atualizando informações do banco de dados postgres**
```bash
superset db upgrade
```
**CRIAR USUÁRIO ADMINISTRADOR  E METADADOS DO SUPERSET NO POSTGRES**
```bash
superset fab create-admin
```
**CARREGANDO ALGUNS EXEMPLOS EXEMPLOS ( OPCIONAL)**
```bash
superset load_examples
```
**CRIAR FUNÇÕES  E PERMISSÕES PADRÕES**
```bash
superset init
```
**CRIANDO ACESSO HTTP DO SUPERSET**
``` bash
gunicorn -w 10  -k gevent --timeout 120 -b  0.0.0.0:8081 --limit-request-line 0 --limit-request-field_size 0 --statsd-host localhost:8088 "superset.app:create_app()"

```

