# Rede social local para comunidade universitária

## 1. Contexto
Sistema: Rede social local para a comunidade universitária, onde alunos e professores podem compartilhar posts, imagens, eventos e interagir por meio de curtidas e comentários. O sistema possui cerca de 10.000 usuários cadastrados, onde pode atingir picos de até 2.000 acessos em 10 minutos em períodos de grande movimentação, como divulgação de eventos ou início de semestre/mátricula.

Funções:
- Criar posts com textos, imagens e links
- Curtir e comentar publicações
- Divulgar eventos universitários
- Troca de mensagens com outros usuarios

## 2. Requisitos
CPU: Processador Multi-core 2 vCPUs no backend para lidar com requisições concorrentes e não bloquear a Main Thread.
Memória (RAM): 8GB. 
Armazenamento: HDD mecânico 500GB
SO: Linux (Ubuntu Server) 


## 3. Diagnóstico da Falha

- Incidente 1 – Erro 404 na Nuvem (Arquivos)

Sintoma:
No feed da rede social, as imagens dos posts aparecem no computador do desenvolvedor, mas no servidor Linux (produção) ficam quebradas.

Análise:
No servidor Linux, o nome dos arquivos e os caminhos precisam ser exatamente iguais aos usados no código. Também é necessário utilizar caminhos relativos para evitar problemas entre diferentes sistemas operacionais.

Causa raiz:
Uso de caminho absoluto do Windows ou diferença entre letras maiúsculas e minúsculas, já que o Linux é *case-sensitive*.

- Incidente 2 – Acesso Negado 403

Sintoma:
O usuário tenta baixar uma imagem ou arquivo da rede social, mas o sistema retorna erro 403 (acesso negado).

Análise:
O servidor está tentando salvar ou acessar arquivos em uma pasta onde não possui permissão de escrita.

Causa raiz:
O servidor não tem permissão de leitura ou escrita na pasta utilizada pela aplicação.


- Incidente 3 – Falta de DNS

Sintoma:
A equipe comprou o domínio redesocialefg.com.br, mas quando os usuários tentam acessar pelo nome, aparece a mensagem “Servidor não encontrado”, embora funcione quando digitam o endereço IP.

Análise:
O serviço de DNS precisa estar configurado e apontando corretamente para o endereço IP do servidor.

Causa raiz:
O domínio foi comprado, mas não foi configurado para direcionar o nome do site para o IP do servidor.


- Incidente 4 – Timeout no Ônibus

Sintoma:
O aplicativo funciona normalmente no Wi-Fi da faculdade, mas no 3G trava na tela de carregamento sem informar nada ao usuário.

Análise:
A baixa velocidade e a menor largura de banda da rede móvel causam alta latência. Isso gera timeout nas requisições e o aplicativo não possui tratamento adequado para informar ao usuário sobre a demora ou falha de conexão.

Causa raiz:
O desenvolvedor não considerou a latência da rede 3G na otimização do código, nem aplicou compressão de imagens e arquivos para reduzir o tempo de resposta. Também não foi implementado tratamento de erro para timeout.


- Incidente 5 – A Panela Derretendo

Sintoma:
A tela do perfil contendo muitas imagens em alta resolução faz os celulares de entrada esquentarem muito, a navegação fica lenta e o aplicativo fecha sozinho.

Análise:
O alto uso de CPU para renderizar as imagens causa aquecimento no dispositivo. Além disso, as imagens grandes são descomprimidas ao mesmo tempo, consumindo muita memória RAM e deixando o aplicativo lento.

Causa raiz:
O desenvolvedor não aplicou compressão e otimização das imagens e também não utilizou lazy loading, que permite carregar imagens apenas quando necessário.


- Incidente 6 – A Interface Congelada

Sintoma:
Ao pesquisar algum usuário ou aplicar filtros, a tela do celular congela por cerca de 4 segundos e os botões não respondem.

Análise:
A interface está rodando na thread principal, e as funções de busca e filtros estão demorando muito tempo para serem processadas.

Causa raiz:
O desenvolvedor não otimizou a função JavaScript e não utilizou programação assíncrona para executar tarefas pesadas fora da *main thread*.


- Incidente 7 – Amnésia Digital

Sintoma:
O usuário começa a escrever uma postagem, minimiza o aplicativo para olhar o WhatsApp e, ao voltar, o app recarrega e todo o conteúdo digitado é perdido.

Análise:
O aplicativo não está armazenando localmente ou no servidor o conteúdo digitado pelo usuário.

Causa raiz:
Não existe um sistema de persistência de dados para salvar rascunhos da postagem, seja em armazenamento remoto no servidor ou armazenamento local no dispositivo (como cache do navegador ou banco de dados local, por exemplo SQLite).


- Incidente 8 – Vazamento de Memória

Sintoma:
O aplicativo possui um feed infinito. Quanto mais o usuário rola a tela, mais lento o celular fica até o aplicativo travar após alguns minutos.

Análise:
O aplicativo está acumulando na memória RAM todo o conteúdo já exibido no feed e não está liberando a memória que não é mais necessária.

Causa raiz:
Má gestão de memória pelo desenvolvedor, que não programou a liberação dos dados antigos da memória, causando memory leak.


- Incidente 9 – Gargalo de Disco

Sintoma:
Durante picos de acesso, o sistema fica lento mesmo com a CPU pouco utilizada.

Análise:
O servidor demora para ler e gravar dados no armazenamento em disco rigido HDD.

Causa raiz:
O uso de disco HDD causa lentidão nas operações de leitura e escrita.


- Incidente 10 – Conflito de Deploy

Sintoma:
O sistema funciona no computador do desenvolvedor, mas apresenta erro ao ser executado no servidor.

Análise:
A versão da linguagem ou das dependências no servidor é diferente da utilizada no ambiente de desenvolvimento, provavelmente porque não foi utilizado um docker para garantir as mesmas configurações.

Causa raiz:
Diferença entre o ambiente de desenvolvimento e o ambiente do servidor.


## 4. Arquitetura Proposta
Ambiente: Em vez de um servidor Bare-metal monolítico, usaremos Containers (Docker) na Nuvem com um Load Balancer (Balanceador de Carga).
Justificativa: 
1) Isolamento: Docker garante o mesmo ambiente do desenvolvimento.
2) Escalabilidade: O Load Balancer distribui os alunos entre 5 containers menores. Se a RAM de um subir, cria-se outro.
3) Otimização: Arquivos estáticos servidos via Cache, poupando processamento de CPU do servidor principal.
- Upgrade de memória (RAM) para 16GB. Mais espaço para o armazenamento durante o processamento das funcionalidades do app. Necessário para evitar que o SO acione a Memória Virtual (Swap) durante o pico.
- Upgrade para SSD NVMe evitar gargalos de I/O na gravação das matrículas e leitura de dados do usuário.
- Upgrade para 2TB de armazenamento SSD para gravar rascunhos de postagens aumentando a persistência dos dados inserido pelo usuário
- Otimização e tratamento de arquivos de imagens para formato WEBP, padronizando tamanhos para evitar sobrecarga de download de dados e processamento de imagens
- Otimização de do codigo para implementar tratamento de erros, gestão de memória com liberação dos dados antigos da memória, programação assíncrona para executar tarefas pesadas fora da *main thread*, lazy loading, que permite carregar imagens apenas quando necessário.







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
