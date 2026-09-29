# natas-overthewire
Relatório Técnico de Segurança Web: Natas 0 ao 7
Este documento regista o meu progresso prático no wargame Natas, focado na aprendizagem de segurança ofensiva em aplicações web. O meu objetivo principal com estes exercícios é consolidar conceitos sobre a arquitetura do protocolo HTTP, análise de código-fonte no cliente e no servidor, manipulação de tráfego com proxy e exploração de falhas na inclusão de ficheiros.
Níveis 0 a 3: Reconhecimento e Exposição de Informações
 Natas 0 (Exposição de Informações): Encontrei as credenciais de acesso escondidas dentro de comentários no próprio código-fonte HTML da página, acessível pelo navegador.
 Natas 1 (Restrições no Lado do Cliente): O site tentou bloquear o uso do botão direito do rato. Contornei a limitação usando o atalho de teclado do navegador (Ctrl + U) para abrir e analisar o código-fonte diretamente.
 Natas 2 (Listagem de Diretórios): Analisei os caminhos de ficheiros estáticos da página e notei a imagem carregada em files/pixel.png. Acedi diretamente à pasta /files/, que estava com a listagem de diretórios aberta, e encontrei o ficheiro users.txt com os acessos.
 Natas 3 (Vazamento via Robots.txt): Acedi ao ficheiro /robots.txt para verificar quais rotas o site tentava esconder dos motores de busca. Encontrei a diretiva para o diretório /s3cr3t/, naveguei até ele e localizei o ficheiro de credenciais.
Níveis 4 e 5: Interceção e Manipulação de Tráfego HTTP
 Natas 4 (Adulteração de Cabeçalhos HTTP): Utilize o Burp Suite para capturar a requisição enviada ao servidor. Modifiquei o cabeçalho Referer para [http://natas5.natas.labs.overthewire.org/](http://natas5.natas.labs.overthewire.org/), fazendo o servidor acreditar que a navegação tinha origem na página exigida.
 Natas 5 (Gestão Insegura de Sessão): Interceptei o tráfego no Burp Suite e identifiquei um cookie de sessão simples em texto limpo. Alterei o valor de loggedin=0 para loggedin=1 e reenviei o pacote, contornando o controlo de autenticação do sistema.
Níveis 6 e 7: Análise no Servidor e Inclusão de Ficheiros
 Natas 6 (Exposição de Ficheiro Interno e Análise de Código):  Análise: Inspecionei o código-fonte PHP disponibilizado na página e vi a instrução include "includes/secret.inc";.
   Exploração: Como o ficheiro incluído estava no diretório público do servidor, acedi diretamente à rota /includes/secret.inc pelo navegador para ler a variável da palavra-chave e submetê-la no formulário.
   Conceito: Ficheiros de configuração que armazenam segredos ou chaves de acesso não podem ficar expostos em pastas acessíveis publicamente na web.  Natas 7 (Inclusão de Ficheiro Local - LFI e Path Traversal):
Análise: Notei que a aplicação carregava o conteúdo das páginas usando um parâmetro na URL (index.php?page=home). Ao olhar o código PHP, percebi que o valor da variável page era passado para uma função de inclusão sem qualquer tipo de filtragem.
   Exploração: Sabendo que no ambiente Linux do Natas as credenciais do nível seguinte ficam guardadas em /etc/natas_webpass/, explorei a falha de Path Traversal alterando a URL para index.php?page=/etc/natas_webpass/natas8. Isso forçou o servidor a ler o ficheiro do sistema e exibir a senha na minha tela.
Conceito: Permitir que o utilizador controle caminhos de ficheiros carregados pelo servidor sem uma validação rigorosa abre espaço para a leitura arbitrária de ficheiros confidenciais do sistema operativo.
Ferramentas que Utilizei
 Navegador Web: Para inspeção de elementos, navegação por parâmetros na URL e leitura de ficheiros estáticos.
Burp Suite Community Edition: Para capturar, analisar e alterar cabeçalhos e cookies de requisições HTTP em tempo real.
Conhecimentos do Linux: Para entender a estrutura de pastas do sistema operativo (/etc/) e onde os dados ficam armazenados.
Principais Aprendizados Técnicos
 * Confiança Zero no Cliente: Não se deve confiar em nenhum dado enviado pelo navegador (sejam cookies, parâmetros de URL ou cabeçalhos), pois tudo pode ser adulterado antes de chegar ao servidor.
 * Tratamento de Dados no Servidor: Qualquer parâmetro que interaja com o sistema de ficheiros precisa ser validado contra uma lista restrita de valores permitidos para evitar vulnerabilidades como LFI.
 * Leitura de Código: Saber analisar o código PHP no servidor é fundamental para entender a lógica da aplicação, identificar onde estão as falhas de validação e saber exatamente como construir o ataque.
