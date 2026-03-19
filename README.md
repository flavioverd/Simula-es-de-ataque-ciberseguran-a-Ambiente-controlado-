# Simula-es-de-ataque-ciberseguran-a-Ambiente-controlado-
Deesafio do Bootcamp Riachuelo usando medusa 

Projeto: Simulação de Ataques de Força Bruta e Prevenção
1. Configuração do Laboratório (Lab Setup)
Para garantir um ambiente seguro e controlado, utilizamos a virtualização para isolar as atividades de teste de sistemas reais
.
Máquinas Virtuais:
Atacante: Kali Linux (equipada com Medusa e ferramentas de rede)
.
Alvo: Metasploitable 2 (distribuição propositalmente vulnerável)
.
Rede: Ambas as VMs configuradas em modo Placa de rede exclusiva de hospedeiro (Host-only), permitindo comunicação interna sem exposição à internet
.
Conectividade: Validamos a comunicação com o comando ping -c 3 [IP_ALVO] no terminal do Kali para confirmar que o alvo está acessível
.
2. Reconhecimento e Enumeração
Antes de iniciar os ataques, é fundamental mapear o alvo para entender quais serviços estão expostos
.
Varredura de Portas (Nmap): Utilizamos o comando nmap -sV -p 21,80,445 [IP_ALVO] para identificar os serviços de FTP, HTTP e SMB, bem como suas versões específicas, o que é crucial para encontrar falhas conhecidas
.
Enumeração de Usuários (enum4linux): Para o ataque de SMB, utilizamos o enum4linux -a [IP_ALVO] para extrair nomes de contas reais do sistema, evitando desperdiçar tempo com usuários inexistentes
.
3. Execução dos Ataques com Medusa
O Medusa é uma ferramenta de força bruta via rede, rápida e modular, que suporta múltiplos protocolos simultaneamente através de multithreading
.
A. Força Bruta em FTP
O objetivo é testar combinações de login e senha contra o serviço de transferência de arquivos
.
Wordlists: Criamos users.txt e passwords.txt com credenciais comuns (ex: admin, msfadmin)
.
Comando: medusa -h [IP_ALVO] -u users.txt -P passwords.txt -M ftp -t 6
.
Validação: Ao encontrar sucesso, validamos o acesso manualmente com ftp [IP_ALVO]
.
B. Automação em Formulário Web (DVWA)
Simulamos o preenchimento em massa de um formulário de login
.
Análise: Usamos as Ferramentas de Desenvolvedor (F12) no navegador para capturar os nomes dos parâmetros (username, password) e a string de erro "login failed"
.
Comando: medusa -h [IP_ALVO] -u users.txt -P passwords.txt -M http -m PAGE:/dvwa/login.php -m FORM:"username=^USER^&password=^PASS^&Login=Login" -m F:"login failed"
.
C. Password Spraying em SMB
Diferente do brute force comum, testamos uma única senha comum contra todos os usuários reais encontrados anteriormente
.
Comando: medusa -h [IP_ALVO] -U SMB_users.txt -p "123456" -M smbnt
.
Validação: Usamos o smbclient -L [IP_ALVO] -U [USUARIO] para listar os compartilhamentos e confirmar o acesso administrativo
.
4. Medidas de Prevenção e Mitigação
A segurança falha quando o básico é negligenciado
. Para proteger o ambiente corporativo, as seguintes medidas são essenciais:
Autenticação Multifator (MFA): Neutraliza o uso de credenciais vazadas, pois o atacante não terá acesso ao segundo fator
.
Políticas de Senhas Fortes: Implementar requisitos de complexidade e rotação periódica para tornar wordlists inúteis
.
Bloqueio por Tentativas Falhas: Configurar o sistema para bloquear endereços IP após múltiplas falhas, dificultando ataques de spray e brute force
.
Desativação de Serviços: Se protocolos como FTP ou SMB não forem estritamente necessários, eles devem ser desativados ou restritos a redes internas segmentadas
.
