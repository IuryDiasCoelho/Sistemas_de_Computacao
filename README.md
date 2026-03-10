# Sistemas de Computação: O Desafio Final

## Introdução

Este trabalho detalha o planejamento de infraestrutura e o troubleshooting de um sistema de biblioteca digital acadêmica hospedado em Nuvem (Cloud Computing). O foco principal é assegurar a estabilidade e fluidez do acesso ao acervo, especialmente em períodos de alta demanda, como semanas de provas e entregas de TCC.

---

# 1. Infraestrutura e Arquitetura

A base do sistema foi projetada para utilizar recursos de **Cloud Computing**, garantindo que o acervo esteja disponível 24h por dia para os alunos:

**Perfil do usuário:**  
Estudantes universitários utilizando desktops (laboratórios), tablets e celulares via redes 3G/4G/5G.

**Capacidade de Processamento (CPU):**  
Servidor com 8 vCPUs Multi-core para gerenciar o volume de requisições e proteger a Main Thread de travamentos.

**Memória RAM:**  
16 GB para manter processos ativos em memória física, minimizando o uso de Swap e mantendo a performance.

**Armazenamento:**  
SSD NVMe em nuvem para garantir altíssima velocidade de leitura (I/O) de arquivos PDF.

**Sistema Operacional:**  
Linux, selecionado por sua estabilidade e eficiência na gestão de serviços de rede e containers.

---

# 2. Diagnóstico e Troubleshooting

Análise técnica de 10 incidentes críticos afetando a experiência do usuário:

## Incidente 1 (Erro 404 na Nuvem)

**Sintoma:**  
A imagem da capa de um livro aparece normalmente no computador do desenvolvedor, mas fica "quebrada" (Erro 404) quando o site é acessado no servidor Linux da biblioteca.

**Camada Afetada:** Arquivos

**Análise:**  
No computador do desenvolvedor, o código aponta para um endereço fixo e local (ex: `C:\Users\Projeto\capa.jpg`). Quando o site vai para a nuvem (Linux), esse caminho não existe. Além disso, o Linux diferencia letras maiúsculas de minúsculas, então se o arquivo for `Capa.jpg` e o código pedir `capa.jpg`, ele não encontrará **(MDN Web Docs, 2026)**.

**Solução:**

1. Utilizar caminhos relativos ou URLs padronizadas para acessar arquivos da aplicação.
2. Padronizar nomes de arquivos utilizando letras minúsculas para evitar inconsistências no servidor Linux.

---

## Incidente 2 (Acesso Negado 403)

**Sintoma:**  
O aluno clica para gerar o "Comprovante de Reserva" de um livro, mas a operação é interrompida silenciosamente e o arquivo não é baixado.

**Camada Afetada:**  
Sistema Operacional ou Permissão

**Análise:**  
O servidor Linux controla o acesso aos arquivos por meio de permissões de leitura e escrita. Caso a aplicação tente criar ou salvar arquivos em um diretório sem permissão adequada, o sistema operacional bloqueia a operação, gerando erro de acesso negado **(TANENBAUM, 2015; MAZIERO, 2019)**.

**Solução:**

1. Ajustar as permissões do diretório de destino no servidor Linux.
2. Garantir que o usuário responsável pela execução da aplicação tenha permissões adequadas para leitura e escrita.

---

## Incidente 3 (Falta de DNS)

**Sintoma:**  
O aluno tenta acessar a biblioteca pelo endereço oficial (ex: `www.bibliotecavirtual.com.br`), mas o navegador diz que o "Servidor não foi encontrado". Porém, se ele digitar o número do IP direto, o site abre.

**Camada Afetada:**  
Rede ou Domínio

**Análise:**  
O sistema de DNS (Domain Name System) é responsável por traduzir nomes de domínio em endereços IP. Se o domínio foi alterado recentemente ou configurado incorretamente, o navegador não consegue localizar o servidor correto, impedindo o acesso ao sistema **(W3C, 2024; MDN Web Docs, 2026)**.

**Solução:**

1. Verificar as configurações do DNS no provedor de domínio.
2. Garantir que o domínio esteja corretamente apontando para o IP do servidor em nuvem.

---

## Incidente 4 (Timeout no Ônibus)

**Sintoma:**  
O aplicativo da biblioteca funciona perfeitamente no Wi-Fi da faculdade, mas trava na tela de carregamento infinita quando o aluno tenta acessar via 3G/4G dentro do ônibus, sem exibir nenhuma mensagem de erro.

**Camada Afetada:**  
Rede

**Análise:**  
Conexões móveis apresentam maior latência e instabilidade em comparação ao Wi-Fi. Quando o tempo de resposta do servidor ultrapassa o limite definido pela aplicação (timeout), a requisição pode falhar ou ficar aguardando indefinidamente caso não exista tratamento adequado no código **(W3C, 2024; MDN Web Docs, 2026)**.

**Solução:**

1. Implementar uma mensagem de aviso após alguns segundos de espera (ex: "Sua conexão está lenta, tente novamente").
2. Otimizar o carregamento de imagens e dados para reduzir o tempo de resposta em redes móveis.

---

## Incidente 5 (A Panela Derretendo)

**Sintoma:**  
Ao carregar telas com muitas imagens em alta resolução, o smartphone do estudante aquece excessivamente, a navegação apresenta "engasgos" (lags) e o aplicativo encerra abruptamente (crash).

**Camada Afetada:**  
Software ou Aplicação

**Análise:**  
O problema ocorre porque o aplicativo tenta carregar imagens muito pesadas ao mesmo tempo. Isso gera alto uso de CPU e GPU no dispositivo móvel. O aquecimento do aparelho é consequência do processamento excessivo causado pela falta de otimização do software **(MDN Web Docs, 2026)**.

**Solução:**

1. **Lazy Loading:** carregar imagens apenas sob demanda.
2. Reduzir o tamanho e a resolução das imagens, utilizando compressão e formatos otimizados para dispositivos móveis.
3. Sempre que possível, usar versões adaptadas das imagens para diferentes tamanhos de tela.

---

## Incidente 6 (A Interface Congelada)

**Sintoma:**  
Ao pesquisar ou filtrar livros, a tela do celular "congela" por alguns segundos e os botões param de responder.

**Camada Afetada:**  
Software

**Análise:**  
O processamento da busca está sendo executado na Main Thread, responsável também por atualizar a interface gráfica. Quando operações pesadas são executadas nessa thread, a interface deixa de responder temporariamente aos comandos do usuário **(MDN Web Docs, 2026)**.

**Solução:**

1. Executar operações de busca e processamento em **threads de segundo plano**.
2. Otimizar o algoritmo de pesquisa e filtragem para reduzir o volume de processamento em cada interação.

---

## Incidente 7 (Amnésia Digital)

**Sintoma:**  
O aluno está preenchendo um formulário de reserva de livro e minimiza o aplicativo para copiar uma informação no WhatsApp. Ao retornar, o aplicativo da biblioteca recarrega do zero e todos os dados preenchidos foram perdidos.

**Camada Afetada:**  
Software ou Sistema Operacional

**Análise:**  
O problema ocorre porque o aplicativo não salva o estado do formulário enquanto o usuário digita. Quando o sistema operacional encerra o aplicativo em segundo plano para liberar memória, os dados não são preservados **(TANENBAUM, 2015; MAZIERO, 2019)**.

**Solução:**

1. Implementar **salvamento automático de estado** do formulário enquanto o aluno preenche os dados.
2. Restaurar automaticamente os dados salvos quando o usuário retornar ao app.

---

## Incidente 8 (Vazamento de Memória)

**Sintoma:**  
O aplicativo da biblioteca possui um catálogo infinito de livros. Quanto mais o aluno rola a tela, mais lento o celular fica, até que o aplicativo fecha sozinho após cerca de 10 minutos.

**Camada Afetada:**  
Software

**Análise:**  
O aplicativo está carregando novos elementos do catálogo continuamente na memória sem liberar os objetos que não estão mais visíveis. Esse comportamento gera um **memory leak**, aumentando progressivamente o uso de RAM até causar encerramento do aplicativo **(MDN Web Docs, 2026)**.

**Solução:**

1. Liberar corretamente objetos que não estão mais em uso.
2. Implementar virtualização de listas, carregando apenas os elementos visíveis.

---

## Incidente 9 (Gargalo de Disco)

**Sintoma:**  
Em horários de pico (entregas de TCC ou períodos de prova), o sistema fica extremamente lento para abrir PDFs ou salvar pesquisas.

**Camada Afetada:**  
Armazenamento

**Análise:**  
Durante períodos de grande acesso simultâneo ao acervo digital, o servidor precisa realizar múltiplas operações de leitura de arquivos PDF. Mesmo utilizando SSD NVMe, o alto volume de requisições pode gerar congestionamento de operações de I/O **(STALLINGS, 2017; HENNESSY; PATTERSON)**.

**Solução:**

1. Utilizar cache para arquivos mais acessados otimizando o acesso ao armazenamento reduzindo leituras repetidas.
2. Distribuir melhor as requisições de leitura durante períodos de pico.
3. Ter um monitoramento do uso de I/O para identificar mais gargalos e ajustar a infraestrutura quando necessário.

---

## Incidente 10 (Conflito de Deploy)

**Sintoma:**  
O programador criou uma função nova que funciona no computador dele, mas no servidor oficial o sistema inteiro para de funcionar.

**Camada Afetada:**  
Software ou Sistema Operacional

**Análise:**  
Diferenças entre o ambiente de desenvolvimento e o ambiente de produção podem causar incompatibilidade de bibliotecas, dependências ou versões do sistema operacional **(Docker Curriculum, 2024)**.

**Justificativa:**

1. Utilizar **Containers (Docker)** para empacotar código e dependências.
2. Garantir que o **sistema de desenvolvimento** seja igual ao **ambiente de produção**.

---

# 3. Definição do Ambiente de Execução

O sistema da biblioteca digital será executado em um servidor Linux hospedado em nuvem, escolhido por sua estabilidade, bom gerenciamento de serviços e suporte a aplicações web.

Neste projeto, foi escolhida a utilização de containers (Docker) em vez de bare-metal ou máquinas virtuais (VMs). O uso de bare-metal não foi adotado porque exige maior gerenciamento da infraestrutura física e oferece menos flexibilidade para escalar recursos. As máquinas virtuais também não foram priorizadas, pois cada VM precisa executar um sistema operacional completo, o que aumenta o consumo de CPU e memória.

Além disso, como o sistema pretende disponibilizar um grande acervo de livros (muitos deles com centenas de páginas) será necessário escalar principalmente o armazenamento. **Nesse cenário, utilizar bare-metal ou máquinas virtuais pode tornar esse processo mais complexo e mais caro.**

Para evitar problemas como “na minha máquina funciona”, **a aplicação será executada em containers Docker**, que empacotam o código, bibliotecas e dependências necessárias para o funcionamento do sistema. Dessa forma, o mesmo ambiente utilizado no desenvolvimento pode ser reproduzido no servidor.

Com essa abordagem, o sistema se torna mais fácil de manter, mais escalável e mais estável, além de suportar vários acessos simultâneos sem comprometer a disponibilidade do acervo digital para os alunos.

---

## Arquitetura Final de Deploy

O relançamento seguro do sistema baseia-se no seguinte fluxo de execução:

**Camada de Cliente:**  
App/Web otimizado com Lazy Loading e persistência de dados para economizar RAM e CPU dos dispositivos dos alunos.

**Camada de Rede:**  
Domínio configurado com DNS estável e suporte a latências de redes móveis (3G/4G/5G).

**Camada de Aplicação (Docker):**  
O sistema roda dentro de contêineres isolados, garantindo que picos de acesso não derrubem a aplicação principal e que o ambiente seja padronizado.

**Camada de Dados (SSD NVMe):**  
Armazenamento de alta performance para entrega imediata de PDFs, eliminando gargalos de leitura física.

---

![Arquitetura.png](Arquitetura.png)

---

# Referências

- TANENBAUM, Andrew S. Organização Estruturada de Computadores. 6. ed. São Paulo: Pearson, 2013.

- MONTEIRO, Mário A. Introdução à Organização de Computadores. 5. ed. Rio de Janeiro: LTC, 2007.

- STALLINGS, William. Arquitetura e Organização de Computadores. 10. ed. São Paulo: Pearson Universidades, 2017.

- HENNESSY, John; PATTERSON, David. Arquitetura de Computadores: Uma Abordagem Quantitativa. Morgan Kaufmann.

- TANENBAUM, Andrew S.; BOS, Herbert.Sistemas Operacionais Modernos. 4. ed. São Paulo: Pearson Universidades, 2015.

- MAZIERO, Carlos A. Sistemas Operacionais: Conceitos e Mecanismos. Curitiba: DINF – UFPR, 2019. Disponível em:
  https://wiki.inf.ufpr.br/maziero/doku.php?id=socm:start

- W3C. Web Standards and Web Architecture. Disponível em: https://www.w3.org

- MDN Web Docs. Web Performance e Ambientes de Execução. Disponível em: https://developer.mozilla.org

- MDN Web Docs. HTTP Overview. Disponível em: https://developer.mozilla.org/pt-BR/docs/Web/HTTP

- Ubuntu Documentation. Command Line for Beginners. Disponível em: https://ubuntu.com/tutorials/command-line-for-beginners

- Docker Curriculum. Docker Tutorial Oficial. Disponível em: https://docker-curriculum.com
