# Rede social local para comunidade universitária

## 1. Contexto
Sistema: Rede social local para comunidade universitária (Total de usúarios 10.000. Picos de 2.000 acessos em 10 min).

## 2. Requisitos
CPU: Processador Multi-core 2 vCPUs no backend para lidar com requisições concorrentes e não bloquear a Main Thread.
Memória (RAM): 8GB. 
Armazenamento: HDD mecânico 500GB
SO: Linux (Ubuntu Server) 


## 3. Diagnóstico da Falha


Incidentes 1 a 10

Sintoma: 
Análise: 
Causa Raiz:

Incidentes 1 a 10

Sintoma: 
Análise: 
Causa Raiz:





## 4. Arquitetura Proposta
Ambiente: Em vez de um servidor Bare-metal monolítico, usaremos Containers (Docker) na Nuvem com um Load Balancer (Balanceador de Carga).
Justificativa: 
1) Isolamento: Docker garante o mesmo ambiente do desenvolvimento.
2) Escalabilidade: O Load Balancer distribui os alunos entre 5 containers menores. Se a RAM de um subir, cria-se outro.
3) Otimização: Arquivos estáticos servidos via Cache, poupando processamento de CPU do servidor principal.
- Upgrade de memória (RAM) para 16GB. Necessário para evitar que o SO acione a Memória Virtual (Swap) durante o pico.
- Upgrade para SSD NVMe evitar gargalos de I/O na gravação das matrículas.







## Exemplo:
1. Contexto e 2. Requisitos
Sistema: Web Acadêmico para matrícula (Picos de 3.000 acessos em 10 min).
CPU: Processador Multi-core (ex: 4 a 8 vCPUs) no backend para lidar com requisições concorrentes e não bloquear a Main Thread.
Memória (RAM): 8GB a 16GB. Necessário para evitar que o SO acione a Memória Virtual (Swap) durante o pico.
Armazenamento: HDD mecânico
SO: Linux (Ubuntu Server)

3. Diagnóstico da Falha
Sintoma: Site carrega infinitamente e exibe erro 503 Service Unavailable.
Análise: A CPU bateu 100% e a RAM esgotou. O problema não é código nem rede do aluno. O gargalo é na interação SO vs Hardware.
Causa Raiz: O servidor web tentou abrir um processo/thread para cada um dos 3.000 alunos. O SO não conseguiu gerenciar a concorrência (troca de contexto da CPU) para tantos processos simultâneos. A RAM acabou e o sistema travou.

4. Arquitetura Proposta
Ambiente: Em vez de um servidor Bare-metal monolítico, usaremos Containers (Docker) na Nuvem com um Load Balancer (Balanceador de Carga).
Justificativa: 1) Isolamento: Docker garante o mesmo ambiente do desenvolvimento.
2) Escalabilidade: O Load Balancer distribui os alunos entre 5 containers menores. Se a RAM de um subir, cria-se outro.
3) Otimização: Arquivos estáticos servidos via Cache, poupando processamento de CPU do servidor principal.
- Upgrade de memória (RAM) para 16GB. Necessário para evitar que o SO acione a Memória Virtual (Swap) durante o pico.
- Upgrade para SSD NVMe evitar gargalos de I/O na gravação das matrículas.

Incidentes 1 a 3: Arquivos, Redes e DNS
Incidente 1 (Erro 404 na Nuvem): Um arquivo ou imagem importante funciona perfeitamente no PC do desenvolvedor, mas quebra quando o site vai para o servidor Linux. (Dica: C:/Users... ou case-sensitive).
Incidente 2 (Acesso Negado 403): O sistema tenta gerar um comprovante ou salvar um log no servidor para o usuário baixar, mas a operação é bloqueada silenciosamente.
Incidente 3 (Falta de DNS): A equipe comprou o domínio .com.br, mas quando os primeiros usuários tentam acessar pelo nome, a página diz "Servidor não encontrado", embora funcione se digitarem o IP.
Incidentes 4 a 6: Mobile e Hardware
Incidente 4 (Timeout no Ônibus): O app funciona bem no Wi-Fi da faculdade. Mas no 3G, o app trava na tela de carregamento sem avisar nada ao usuário.
Incidente 5 (A Panela Derretendo): Uma tela com muitas imagens em alta resolução faz os celulares de entrada esquentarem muito, a navegação fica lenta e o app fecha sozinho.
Incidente 6 (A Interface Congelada): Ao pesquisar algo ou aplicar um filtro, a tela do celular "congela" por 4 segundos. Os botões não respondem aos toques.
Incidentes 7 a 10: SO, Concorrência e Deploy
Incidente 7 (Amnésia Digital): O usuário preenche metade de um formulário e minimiza o app para olhar o WhatsApp. Ao voltar, o app recarrega do zero e ele perde tudo.
Incidente 8 (Vazamento de Memória): O app possui um feed infinito. Quanto mais o usuário rola a tela, mais lento o celular fica, até o aplicativo dar Crash após 10 minutos.
Incidente 9 (Gargalo de Disco): O servidor na nuvem é barato e usa disco HDD. Em momentos de pico, o processador (CPU) fica ocioso, mas o sistema fica muito lento para ler/gravar dados.
Incidente 10 (Conflito de Deploy): O código foi feito usando uma versão moderna de uma linguagem, mas o servidor da nuvem tem uma versão antiga instalada, quebrando o sistema inteiro ao tentar subir o código.
