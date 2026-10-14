# Continuidade — MuServer Recriado / Engenharia Reversa estática

> **Documento mestre de retomada.** Leia este arquivo antes de agir em qualquer novo chat. Atualize-o no GitHub após cada etapa confirmada, usando somente evidências realmente observadas. A fonte oficial desta continuidade é este repositório. O estado local pode ter avançado depois da última atualização; confira os artefatos antes de repetir ou continuar qualquer lote.

## 1. Objetivo geral

Catalogar e compreender, por análise estática, os componentes do MuServer e suas relações; construir documentação baseada em evidências; depois planejar uma implementação independente dos sistemas necessários, com testes. O escopo é **o MuServer inteiro**, não apenas ConnectServer ou GameServer:

- executáveis EXE;
- bibliotecas DLL;
- scripts, configurações e tabelas textuais;
- relações entre binários, configurações e protocolos;
- inventário, cobertura, falhas e pendências.

Não considerar o trabalho concluído apenas porque um arquivo foi importado ou porque um decompilador produziu pseudocódigo. Separar sempre: **observação confirmada**, **inferência com confiança indicada** e **hipótese a verificar**.

## 2. Regras obrigatórias de segurança e preservação

- Análise estática apenas: **não executar os binários originais do servidor**.
- Não contornar licença, DRM, anti-tamper, Themida ou outros mecanismos de proteção. Não fazer bypass de controles de acesso.
- Preservar os arquivos originais e todas as evidências já produzidas.
- Não usar comandos destrutivos como `git reset --hard`, `git clean -fd`, remoções em massa ou sobrescrita de resultados existentes.
- Não alterar os bytes dos binários-alvo.
- Não sobrescrever resultados de decompilação existentes sem cópia e justificativa explícita; preferir novos nomes/versionamento.
- Executar Ghidra/REA **sequencialmente**: não iniciar outro lote enquanto houver processamento ativo. O processo MCP do REA, por si só, não significa que exista decompilação ativa.
- Não classificar uma função/binário como analisado até reconciliar inventário, resultados, erros e cobertura.
- Preservar logs de erro completos; resultados parciais são evidência, não sucesso integral.
- Não subir binários proprietários, licenças, IDs de hardware, credenciais, segredos, configurações sensíveis nem grandes dumps proprietários ao GitHub. Publicar apenas documentação segura, scripts próprios e evidências textuais que possam ser compartilhadas legalmente. Confirmar direitos/licença antes de publicar pseudocódigo decompilado.
- Antes de qualquer mudança destrutiva ou mudança de escopo, parar e pedir autorização.
- Nunca afirmar que algo foi salvo no GitHub sem confirmar o resultado da operação/commit.

## 3. Repositório e regra de continuidade

- Repositório: https://github.com/heldergaraujo2/MuserverRecriado
- Arquivo mestre: [PROJECT_MEMORY/REVERSE_ENGINEERING_CONTINUITY.md](https://github.com/heldergaraujo2/MuserverRecriado/blob/main/PROJECT_MEMORY/REVERSE_ENGINEERING_CONTINUITY.md)
- Branch observada: `main`.
- Atualização confirmada mais recente deste documento: preencher pelo SHA retornado após o commit.
- Após cada resultado trazido pelo usuário: atualizar estado, evidências, contagens, falhas e **próxima ação exata** neste arquivo; então confirmar o SHA do commit. Manter o histórico anterior relevante, não inventar dados ausentes.
- Em um novo chat, primeiro ler este arquivo e conferir o estado real dos artefatos citados. Se o documento e o disco divergirem, não apagar nada: registrar a divergência e reconciliar.

## 4. Ambiente local conhecido

- Raiz do MuServer: `D:\Muserver-GitHub\MuServer - Copia`
- GameServer: `D:\Muserver-GitHub\MuServer - Copia\GameServer\GameServer.exe`
- ConnectServer: `D:\Muserver-GitHub\MuServer - Copia\ConnectServer\ConnectServer.exe`
- REA pacote de evidências: `D:\REA\pacote_muserver`
- Inventário global: `D:\REA\analise_MuServer`
- Fila global: `D:\REA\analise_MuServer\fila_binarios.csv`
- Inventário de binários sem cópias: `D:\REA\analise_MuServer\inventario_sem_copias.csv`
- Inventário de duplicatas: `D:\REA\analise_MuServer\duplicatas_binarios.csv`
- Fila priorizada: `D:\REA\pacote_muserver\fila_priorizada.csv`
- Plano de análise: `D:\REA\pacote_muserver\PLANO_ANALISE_BINARIOS.md`
- Node: `C:\Program Files\nodejs\node.exe`
- CLI REA: `D:\REA\npm-cache\_npx\dea9475797100b20\node_modules\rea-agents\scripts\rea.mjs`
- Raiz do pacote REA: `D:\REA\npm-cache\_npx\dea9475797100b20\node_modules\rea-agents`
- Ghidra: `D:\REA\tools\ghidra\ghidra_12.1.4_PUBLIC`
- Ghidra headless: `D:\REA\tools\ghidra\ghidra_12.1.4_PUBLIC\support\analyzeHeadless.bat`
- Java previamente observado: `D:\REA\tools\jdk\jdk-25.0.4.1+1\bin\java.exe`
- Scripts Ghidra próprios: `D:\REA\work\ghidra-scripts`
- Projeto Ghidra isolado anterior: `D:\REA\work\muserver-analysis-clean`, projeto `MuGameServerClean`
- Projeto Ghidra antigo a preservar: `D:\REA\work\muserver-analysis`
- Cópia local do servidor comparada em etapa anterior com a árvore rastreada: 9.514 arquivos e 9.514 hashes SHA-256 comparados idênticos naquele momento. Não presumir que o estado atual continue idêntico sem nova verificação.
- Verificar versões instaladas antes de depender de versões antigas: REA 6.4.0 e JDK 25.0.4.1 foram usados anteriormente.

### Forma de chamar o REA

Ajuda observada:
```powershell
& $node $cli decompile <caminho-do-binario> --procedure 0xADDRESS --provider ghidra --format json --filter-output normalized_result
```

- O CLI oferece `decompile`, `search`, entre outros comandos.
- Uma resposta com `--filter-output normalized_result` pode ser um JSON que contém uma **string JSON**, não diretamente o pseudocódigo. Em resultados já observados, a extração foi:
```powershell
$pseudocodigo = ConvertFrom-Json -InputObject (Get-Content -LiteralPath $json -Raw)
if ($pseudocodigo -isnot [string]) { throw "Formato inesperado: esperado JSON contendo string normalizada" }
Set-Content -LiteralPath $fonte -Value $pseudocodigo -Encoding UTF8
```
- Antes de reutilizar qualquer comando de inventário/pesquisa, confirme a sintaxe com `rea search --help` ou a ajuda correspondente.
- Não chamar `call_tool` através de `call_tool`; para a integração MCP, o padrão correto previamente identificado é `call_toolset_tool(toolset, tool, arguments)`.

## 5. Inventário global observado — precisa ser reconciliado

Em uma auditoria anterior foram reportados:
- 4.723 arquivos no inventário;
- 65 EXE/DLL sob os critérios então usados, excluindo `SpeedServer` e `SubSalatest`;
- 47 hashes únicos de binários;
- 18 cópias físicas duplicadas;
- 2.893 arquivos de scripts/configuração;
- fila global com 47 entradas únicas, mas com status inicial possivelmente desatualizado (por exemplo, GameServer aparecia pendente apesar de já haver evidência).

Essas contagens são **históricas, não garantidas como estado atual**. Revalidar arquivos e hashes antes de confiar nelas ou alterar status. Não apagar cópias duplicadas: registrar caminhos e SHA-256; a mesma análise pode ser reutilizada para cópias idênticas, documentando a equivalência.

Três cópias idênticas de GameServer foram observadas anteriormente:
- `GameServer\GameServer.exe`
- `SpeedServer\GameServer\GameServer.exe`
- `SubSalatest\GameServer\GameServer.exe`

A prioridade planejada era: P0 GameServer (já parcialmente analisado); P1 ConnectServer, ConnectServer_old, DataServer, JoinServer; P2 demais EXEs; P3 DLLs, começando por classificação; além de catalogar scripts/configs e correlacioná-los com os binários. Ajustar essa fila com evidência real.

## 6. GameServer — estado parcial confirmado

- Caminho: `D:\Muserver-GitHub\MuServer - Copia\GameServer\GameServer.exe`
- SHA-256: `ACC5CF86001AF2DD93C5BFC2D8F6B11AA2E665A4F7456732A0F02C79D584A461`
- Tamanho: 13.449.744 bytes.
- PE x86 / 32 bits; Ghidra reportou seção chamada `.themida` de cerca de 12 MB. O nome da seção, isoladamente, **não comprova** comportamento ou proteção específica.
- Em uma etapa anterior, Ghidra marcou análise/importação como bem-sucedida; avisos sobre exports sem função e resolução de símbolos foram observados. O servidor não foi executado.
- Projeto Ghidra antigo deve continuar preservado; o projeto isolado `MuGameServerClean` também foi usado anteriormente.

### Artefatos REA/GameServer já reportados

Raiz de evidências: `D:\REA\pacote_muserver\GameServer`

- `inventario_procedimentos.json.txt` (~5.599 bytes em uma listagem).
- `RELATORIO_CONSOLIDADO.md` (~5.188 bytes).
- `indice_funcoes.csv` (~984 bytes).
- `relatorio_decompilacao.csv` (~23.711 bytes).
- Validação anterior: 13 seções de função, 13 blocos de código C, 24 ocorrências de chamadas `FUN_*`, e `entry` presente.
- 20 símbolos totais foram listados: 7 externos e 13 internos:
  - `entry` — `0x2c576f9`
  - `FUN_02c576ff` — `0x2c576ff`
  - `thunk_FUN_02ddd8fe` — `0x2d626c9`
  - `FUN_02d9d173` — `0x2d9d173`
  - `FUN_02dd1fa2` — `0x2dd1fa2`
  - `FUN_02dd1fb6` — `0x2dd1fb6`
  - `FUN_02dd1fc0` — `0x2dd1fc0`
  - `FUN_02ddd8fe` — `0x2ddd8fe`
  - `FUN_02deb620` — `0x2deb620`
  - `FUN_02e5aca9` — `0x2e5aca9`
  - `FUN_02e6f72b` — `0x2e6f72b`
  - `FUN_02ec276c` — `0x2ec276c`
  - `FUN_02f1f4ad` — `0x2f1f4ad`
- 12 decompilações foram reportadas como bem-sucedidas; uma limpeza/finalização falhou para `FUN_02ddd8fe`, com resultado parcial salvo em `funcoes\FUN_02ddd8fe.c.txt`:
  ```c
  void FUN_02ddd8fe(void) {
    FUN_02d9d173();
    return;
  }
  ```
  Erro completo em `funcoes\FUN_02ddd8fe.erro.txt` (~22.610 bytes). Não marcar como sucesso integral.
- Arquivos JSON de função reportados: `entry`, `FUN_02c576ff`, `FUN_02d9d173`, `FUN_02dd1fa2`, `FUN_02dd1fb6`, `FUN_02dd1fc0`, `FUN_02deb620`, `FUN_02e5aca9`, `FUN_02e6f72b`, `FUN_02ec276c`, `FUN_02f1f4ad`, `thunk_FUN_02ddd8fe`.
- Grafo parcial observado:
  - `entry -> FUN_02dd1fb6`
  - `FUN_02dd1fb6 -> FUN_02e6f72b` e `FUN_02c576ff`
  - `FUN_02c576ff -> FUN_02deb620`
  - `FUN_02deb620 -> FUN_02dd1fc0`
  - `FUN_02dd1fc0 -> FUN_02ec276c` e `FUN_02dd1fa2`
  - `FUN_02dd1fa2 -> FUN_02e5aca9`
- Algumas funções usam `LOCK()/UNLOCK()`; `FUN_02f1f4ad` tem despacho indireto complexo/obfuscado e avisos do Ghidra. Semântica ainda não estabelecida.
- Imports reportados: `KERNEL32.DLL::GetModuleHandleA`, `USER32.DLL::SetTimer`, `GDI32.DLL::GetStockObject`, `ODBC32.DLL::SQLSetEnvAttr`, `WS2_32.DLL::WSASocketA`, `RPCRT4.DLL::UuidCreateSequential`, `PSAPI.DLL::GetModuleInformation`.
- Uma análise anterior do Ghidra mencionou exportação com 13 funções/208 strings e strings relacionadas a WinLicense; confirmar nos arquivos locais se necessário. Não deduzir comportamento completo a partir de strings.

## 7. ConnectServer — inventário confirmado; cobertura ainda incompleta

- Executável: `D:\Muserver-GitHub\MuServer - Copia\ConnectServer\ConnectServer.exe`
- SHA-256: `F25EECB6C65214B7658FC2A8A1B0C786BBB1B718A78405A8E4C61D88C9477AED`
- Tamanho: 380.416 bytes; data de modificação observada: 28/02/2026 13:30:49.
- Pasta de evidências: `D:\REA\pacote_muserver\ConnectServer`
- Pastas de funções/logs: `funcoes` e `diagnostico`.
- Inventário: `diagnostico\inventario_procedimentos.json.txt` (~81.122 bytes), com JSON validado, 332 itens normalizados: 140 imports externos e 192 entradas não externas. As 192 entradas precisam ser classificadas/reconciliadas.
- Log do inventário: `diagnostico\execucao.log`; execução reportada como concluída em 2026-10-14T15:39:28.
- CSVs: `procedimentos_internos.csv` (~5.681 bytes), `simbolos_externos.csv` (~6.606 bytes), `resumo_inventario.txt` (~2.778 bytes).
- Catálogo: `manifesto_evidencias.csv` (~3.424 bytes, versão inicial), `manifesto_evidencias_v2.csv` (~6.410 bytes), `RELATORIO_EVIDENCIAS.md` e `RELATORIO_EVIDENCIAS_v2.md` (~750 bytes).
- O catálogo v2 reportou 29 arquivos, 9 pseudocódigos, 10 resultados JSON do REA, 10 logs e 0 arquivos de erro catalogados naquele catálogo. Isso não significa que todo ConnectServer esteja analisado.
- Verificação anterior encontrou faltando os `.c.txt` de `FUN_00401030` e `FUN_00401060`; ambos foram recuperados de seus JSONs, com tamanhos de 209 e 118 bytes.
- Imports incluem APIs de arquivo, threads, IO completion ports, sincronização, exceções e depuração de KERNEL32. O inventário inclui, entre outros, `GetLocalTime`, `CreateFileA`, `ReadFile`, `WriteFile`, `CreateThread`, `CreateIoCompletionPort`, `GetQueuedCompletionStatus`, `WaitForSingleObject`, `IsDebuggerPresent` e `TerminateProcess`. Imports sugerem áreas a investigar, não provam por si só o fluxo de execução.
- Inferências parciais previamente anotadas:
  - `FUN_00401030`: destrutor semelhante a C++, ajusta vtable para `CClientManager::vftable` e pode chamar `operator_delete(this)`.
  - `FUN_00401060`: inicializa o primeiro campo com `CClientManager::vftable`; provável auxiliar de construção.
  - `FUN_00401070`: alocação/inicialização de campos, chama `GetTickCount`, `FUN_00401960` e `FUN_00405860` com código `0x10004c1`; há registradores não resolvidos, portanto sem conclusão firme.
  - `FUN_00401960`: parece localizar/atualizar/registrar entrada por chave textual; semântica ainda incerta.
  - `FUN_00401ED0`: compara chave textual e retorna ponteiro.
  - `FUN_00402150`: parece percorrer árvore ordenada usando comparação de bytes; estrutura exata incerta.
  - `FUN_00402840`: comparador de sequência de bytes; argumento de comprimento não resolvido (`in_EAX`).
  - `FUN_004022C0`: provável busca e possível inserção; chama comparador e `FUN_004024A0`.
  - `FUN_004024A0`: evidência estrutural forte de inserção/rebalanceamento de árvore rubro-negra de `std::map/set` (ponteiros pai/filhos, flags de cor, rotações e mensagem “map/set<T> too long”). Chaves/dados concretos ainda desconhecidos.
- Endereços iniciais da lista não externa incluem `0x401030 FUN_00401030`, `0x401060 FUN_00401060`, `0x401070 FUN_00401070`, `0x4011a0 FUN_004011a0`, `0x4014a0 FUN_004014a0`, `0x401520 FUN_00401520`, `0x401620 FUN_00401620`, `0x401660 FUN_00401660`, `0x401720 FUN_00401720`, `0x4017c0 FUN_004017c0`, `0x4017f0 FUN_004017f0`, `0x401810 FUN_00401810`, `0x401890 FUN_00401890`, `0x401960 FUN_00401960`, `0x401b40 FUN_00401b40`, `0x401b70 FUN_00401b70`, `0x401c40 FUN_00401c40`, `0x401ed0 FUN_00401ed0`, `0x401f60 FUN_00401f60`, `0x401fc0 FUN_00401fc0`, `0x402020 FUN_00402020`, `0x402040 FUN_00402040`, `0x402090 FUN_00402090`, `0x402150 FUN_00402150`, `0x402250 FUN_00402250`, `0x4022c0 FUN_004022c0`, `0x4024a0 FUN_004024a0`, `0x4026d0 FUN_004026d0`, além de entradas como `Catch_All@0040277d` e `Catch_All@00402b83`.
- Observação sobre catálogo: entradas de exceção como `Catch_All@...` podem não obedecer ao padrão de nome `FUN_...`; o script de lote não deve descartá-las sem classificação.

## 8. Lote-piloto ConnectServer — concluído com 1 falha (2026-10-14)

O usuário compartilhou o resumo do lote executado em 2026-10-14 às 16:22:40 -03:00. O resumo reporta que o executável permaneceu inalterado durante o lote.

- SHA-256 inicial: `F25EECB6C65214B7658FC2A8A1B0C786BBB1B718A78405A8E4C61D88C9477AED`
- SHA-256 final: `F25EECB6C65214B7658FC2A8A1B0C786BBB1B718A78405A8E4C61D88C9477AED`
- Executável inalterado: `True` (conforme resumo do lote).
- Funções tentadas: 10; sucessos reportados: 9; falhas reportadas: 1; conflitos preservados: 0.
- CSV: `D:\REA\pacote_muserver\ConnectServer\resultado_lote_20261014_161609.csv` (4.205 bytes).
- Resumo: `D:\REA\pacote_muserver\ConnectServer\RESUMO_LOTE_20261014_161609.md` (1.611 bytes).
- Logs e eventuais erros: `D:\REA\pacote_muserver\ConnectServer\diagnostico`.
- O usuário informou que os arquivos anteriores não foram apagados.

### Resultado individual do lote

| Função | Endereço | Estado reportado |
|---|---:|---|
| `FUN_004011a0` | `0x4011a0` | SUCESSO |
| `FUN_004014a0` | `0x4014a0` | SUCESSO |
| `FUN_00401520` | `0x401520` | SUCESSO |
| `FUN_00401620` | `0x401620` | SUCESSO |
| `FUN_00401660` | `0x401660` | SUCESSO |
| `FUN_00401720` | `0x401720` | SUCESSO |
| `FUN_004017c0` | `0x4017c0` | FALHA — REA terminou com código 1 |
| `FUN_004017f0` | `0x4017f0` | SUCESSO |
| `FUN_00401810` | `0x401810` | SUCESSO |
| `FUN_00401890` | `0x401890` | SUCESSO |

**Interpretação prudente:** são 9 sucessos conforme validação do script e 1 falha. Ainda não foram inspecionados nesta conversa o conteúdo dos 9 pseudocódigos, o CSV completo nem o log específico de `FUN_004017c0`; portanto, não atribuir semântica às funções nem declarar cobertura integral do lote sem essa inspeção.

### Próximas verificações necessárias

1. Ler a linha de `FUN_004017c0` no CSV e o log/erro correspondente para entender a falha real.
2. Conferir que os 9 resultados marcados como sucesso têm JSON válido, pseudocódigo não vazio e logs associados; validar se resultados anteriores foram preservados em vez de sobrescritos.
3. Auditar a lógica de retomada: a existência de um JSON não deve, por si só, ser tomada como prova de resultado íntegro.
4. Continuar a fila das funções internas do ConnectServer sem ignorar nomes como `Catch_All@...` ou outras entradas que não correspondam ao padrão `FUN_...`.
5. Não iniciar outro processamento REA/Ghidra concorrente se houver algum processo de análise ativo.

**Validação dos nove sucessos (2026-10-14, saída PowerShell fornecida pelo usuário):** foram verificados `FUN_004011a0`, `FUN_004014a0`, `FUN_00401520`, `FUN_00401620`, `FUN_00401660`, `FUN_00401720`, `FUN_004017f0`, `FUN_00401810` e `FUN_00401890`. Os nove arquivos `.json.txt` são JSON válido e desserializam para strings; os nove arquivos `.c.txt` existem e não estão vazios. Contagem: 9/9 JSON válido, 9/9 fontes não vazias, 0 pendências nessa validação. Isso valida a presença/estrutura básica dos artefatos, não a correção semântica do pseudocódigo nem, isoladamente, a integridade de todos os logs.

## 9. Plano de lotes para o MuServer inteiro

1. **Lote 0 — auditoria/preparação:** verificar versões, inventário, hashes, status dos processos e evidências existentes; não sobrescrever.
2. **Lote 1 — ConnectServer:** terminar inventário e cobertura de todas as funções internas; testar reexecução/retomada.
3. **Lote 2 — GameServer:** reconciliar as 13 funções já reportadas, erros, duplicatas e evidências Ghidra/REA; ampliar somente conforme suportado pelas ferramentas e permitido pelas proteções.
4. **Lote 3 — ConnectServer_old.**
5. **Lote 4 — DataServer.**
6. **Lote 5 — JoinServer.**
7. **Lote 6 — demais EXEs prioritários.**
8. **Lote 7 — demais EXEs.**
9. **Lote 8 — DLLs:** primeiro classificar por hash, imports/exports, dependências e duplicatas; depois definir profundidade de análise.
10. **Lote 9 — scripts/configs/tabelas:** inventariar, identificar referências cruzadas e documentar formato/semântica com evidências.
11. **Lote 10 — auditoria global:** cruzar fila, hashes, artefatos, cobertura, erros e dependências; apontar o que falta.
12. **Lote 11 — síntese/reconstrução independente:** escrever especificações e testes de compatibilidade para cada sistema compreendido; só então implementar, evitando copiar código protegido/decompilado.

Os lotes são um plano, não evidência de que foram executados. Registrar estado real por binário/função e continuar em lotes retomáveis, com um manifesto mestre e logs por unidade.

## 10. Estrutura recomendada de evidências locais

Preservar as pastas existentes. Para cada binário, manter ou acrescentar:
- metadados: caminho original, tamanho, timestamp observado, SHA-256, arquitetura/formato;
- inventário de funções/símbolos e imports/exports;
- resultado bruto REA/JSON;
- pseudocódigo textual quando disponível e permitido;
- log por tentativa, status explícito e resumo do erro;
- CSV de resultado por função com endereço, nome, status, caminho dos artefatos, duração se medida, tentativa, observações;
- resumo Markdown do lote com contagens, falhas, itens pulados e próximo passo;
- registro de cobertura reconciliado com o inventário.

Não presumir que todos os artefatos listados existam hoje. Verificar o disco antes de agir. Não subir ao GitHub dumps completos ou pseudocódigo sem confirmar que podem ser publicados.

## 11. Protocolo para cada novo resultado

Após o usuário executar um comando e colar a saída:
1. Interpretar apenas o que a saída realmente comprova.
2. Confirmar arquivos produzidos, tamanho/contagem, status e erros, pedindo saída adicional se faltar evidência.
3. Não iniciar outro processo concorrente.
4. Atualizar este documento: data/hora observada, etapa, comando/ação, resultado, artefatos, falhas/avisos, confiança das inferências e próxima ação exata.
5. Fazer commit da atualização neste repositório e confirmar o SHA retornado.
6. Entregar ao usuário o próximo comando PowerShell completo, com caminhos corretos, evitando comandos destrutivos.
7. Manter o escopo no MuServer inteiro; não encerrar a tarefa após ConnectServer.

## 12. Próxima etapa exata no momento desta atualização

O lote-piloto ConnectServer de 2026-10-14 terminou segundo o resumo fornecido: 10 tentativas, 9 sucessos reportados e 1 falha. O hash do executável antes/depois foi igual. Os artefatos principais foram salvos e o resumo está registrado na seção 8.

**Atualização de diagnóstico (2026-10-14, após inspeção do usuário):** O arquivo `lote_20261014_161609_FUN_004017c0.erro.txt` contém somente `REA terminou com código 1.`; o log relacionado tem 0 bytes. Os arquivos `funcoes\FUN_004017c0.json.txt` e `funcoes\FUN_004017c0.c.txt` estão ausentes. Portanto, a falha não deixou JSON nem pseudocódigo visíveis nos caminhos esperados e a causa técnica ainda não foi determinada. Não repetir ainda a decompilação; primeiro validar os nove sucessos e consultar o help/forma de captura de stdout/stderr para garantir que a próxima tentativa diagnóstica preserve a saída real do processo.\n\n**Próxima ação:** executar uma auditoria somente de leitura dos resultados do lote-piloto, começando por:
- `D:\REA\pacote_muserver\ConnectServer\resultado_lote_20261014_161609.csv`
- `D:\REA\pacote_muserver\ConnectServer\RESUMO_LOTE_20261014_161609.md`
- log/erro específico de `FUN_004017c0` dentro de `D:\REA\pacote_muserver\ConnectServer\diagnostico`.

Pedir ao usuário para executar um comando PowerShell que localize e mostre o log de `FUN_004017c0` e as linhas pertinentes do CSV; não repetir a decompilação dessa função antes de compreender o erro. A validação dos nove sucessos foi concluída: os nove JSONs são válidos e os nove pseudocódigos existem e não estão vazios; nenhuma pendência nessa checagem. Próximo: reconciliar o CSV de resultados com o inventário completo e verificar os logs associados sem reprocessar funções concluídas. Em seguida, criar/rodar um lote sequencial retomável para as funções restantes do ConnectServer, incluindo a classificação de entradas não `FUN_...`, antes de avançar aos próximos binários na fila global.

## 13. Histórico anterior do Ghidra (preservado)

Em 2026-10-13:
- Foi verificado que `D:\REA` não era um repositório Git.
- A cópia `MuServer - Copia` foi comparada com a árvore rastreada do MuServer: 9.514 arquivos e 9.514 hashes SHA-256 comparados idênticos naquela ocasião.
- O GameServer foi importado/analisado no projeto isolado `MuGameServerClean`; logs indicaram “Analysis succeeded”, “Save succeeded” e “Import succeeded”.
- Ghidra identificou PE x86:LE:32:default:windows, sem informação PDB.
- Foram inspecionados os scripts próprios `ExportMuSummary.java`, `ExportMuImports.java` e `ExportMuExports.java`; cada um recebe caminho de saída como argumento.
- Arquivos observados no projeto isolado incluíam `MuGameServerClean.rep`, `MuGameServerClean.gpr` (listado com 0 bytes; isso não prova corrupção), `ghidra-headless.log` e `ghidra-scripts.log`.
- A próxima etapa antiga era exportar o resumo Ghidra, mas o fluxo evoluiu para o inventário global REA descrito acima. Não repetir essa etapa automaticamente; verificar se os relatórios já foram gerados.

## 14. Instrução para um novo chat

Comece lendo este arquivo integralmente. O lote-piloto de ConnectServer já tem resultado reportado na seção 8. Antes de propor qualquer comando, verifique se existe processamento ativo e confira os artefatos locais. Continue do item **12 — Próxima etapa exata**. Um comando por vez, aguarde a saída, mantenha análise estática, preserve tudo, cubra o MuServer inteiro e atualize este documento no GitHub depois de cada resultado confirmado.


## 15. Atualização de continuidade — lote de símbolos internos ConnectServer (2026-10-14)

**Evidência fornecida pelo usuário no terminal local; o conteúdo bruto permanece local e não foi copiado para este repositório.**

- Executável analisado: `D:\\Muserver-GitHub\\MuServer - Copia\\ConnectServer\\ConnectServer.exe`.
- SHA-256 antes e depois do lote: `F25EECB6C65214B7658FC2A8A1B0C786BBB1B718A78405A8E4C61D88C9477AED`; o usuário confirmou `Hash preservado: True`.
- Inventário: `D:\\REA\\pacote_muserver\\ConnectServer\\diagnostico\\inventario_procedimentos.json.txt`. O formato real de cada item em `normalized_result` usa `value` e `address` (não `name`), totalizando 332 itens; uma auditoria anterior usando `name` contou incorretamente zero funções e foi corrigida.
- Funções nomeadas `FUN_<hex>`: a auditoria corrigida deve usar `value`. Depois de recuperar `FUN_004017c0`, há 133 pares de artefatos `.c.txt` e `.json.txt` reportados no diretório `funcoes`. A fila `fila_funcoes_pendentes.csv` tem 114 registros, todos `SUCESSO_VALIDADO`. Isso confirma o estado da fila e presença dos artefatos reportados, não compreensão semântica completa.
- `FUN_004017c0` foi descompilada em nova tentativa usando `--full-output`; a saída JSON era válida, com `normalized_result` não vazio. Foram criados `funcoes\\FUN_004017c0.c.txt` (274 bytes) e `funcoes\\FUN_004017c0.json.txt` (8.548 bytes). Inferência estrutural cautelosa: destrutor de estilo C++ para `CCriticalSection`, com ajuste de vtable, chamada a `DeleteCriticalSection` e desalocação condicional.
- Foi criada a classificação local `diagnostico\\cobertura_20261014\\classificacao_entradas_nao_FUN.csv` com 59 entradas internas que não seguem o padrão `FUN_<hex>`: 4 `HANDLER_EXCECAO`, 2 `PONTO_ENTRADA`, 25 `REVISAR_MANUALMENTE`, 22 `RUNTIME_OU_BIBLIOTECA`, 5 `SEGURANCA_SEH`, 1 `SIMBOLO_CONFLITANTE`. Essas categorias são heurísticas de triagem, não confirmação semântica.
- O lote consolidado de endereços únicos foi executado sequencialmente no diretório local `D:\\REA\\pacote_muserver\\ConnectServer\\diagnostico\\cobertura_20261014\\lote_simbolos_internos_20261014_183049`.
  - Manifesto: `manifesto.csv`.
  - Resultado por entrada: `resultados.csv`.
  - Log: `execucao.log`.
  - 59 entradas candidatas e 59 endereços únicos processados.
  - 57 resultados `SUCESSO_BRUTO_VALIDADO`; 2 falhas; cada entrada tem stdout, stderr e metadados individuais.
  - Falhas: `___tmainCRTStartup` em `0x406b5d` (código de saída 1, stdout 10.580 caracteres) e `MSVCR100.DLL::type_info::_type_info_dtor_internal_method` em `0x40726a` (código 1, stdout 5.306 caracteres). O stdout foi preservado; não foi estabelecida a causa das falhas.
  - `SUCESSO_BRUTO_VALIDADO` significa somente que o comando terminou com código zero e a saída pôde ser validada como JSON com `ok=true` e `normalized_result` não vazio. Não significa que os artefatos finais foram normalizados nem que a semântica foi revisada.
- Nenhum binário, pseudocódigo bruto, licença ou segredo foi adicionado a este commit.

### Próxima ação exata

1. Ler localmente o `resultados.csv`, os dois arquivos `stdout.json.txt` e os respectivos `stderr.txt` das falhas; determinar se o código 1 decorreu de erro de finalização após produzir saída parcial ou de resultado inválido. Não reexecutar os 57 sucessos.
2. Validar os 57 JSONs e extrair as saídas normalizadas em uma nova etapa versionada, preservando todos os arquivos existentes. Antes de criar artefatos oficiais para símbolos não `FUN_`, conferir convenção de nomes, endereço e se a saída é uma função real ou um thunk/auxiliar.
3. Reconciliar as 59 entradas com os 192 itens não externos do inventário e com os 133 pares de artefatos `FUN_`; não declarar cobertura total até documentar a diferença e a classificação de imports/thunks/handlers/runtime.
4. Atualizar os relatórios locais de cobertura e só depois prosseguir à fila global (GameServer, ConnectServer_old, DataServer, JoinServer e demais binários), mantendo análise estática e hash do original.

**Estado do GitHub antes desta atualização:** o arquivo mestre estava no blob SHA `d5d95daf862a424492237d425b1f1c3d4ef9c3c5`, cujo texto ainda descrevia o lote-piloto antigo. Esta seção registra os resultados posteriores fornecidos em 2026-10-14; o SHA do commit que grava esta atualização deve ser o retornado pela operação GitHub.

## 16. Recuperação parcial de falhas de limpeza do Ghidra (2026-10-14)

- Lote de origem: `ConnectServer/diagnostico/cobertura_20261014/lote_simbolos_internos_20261014_183049`.
- As duas entradas com código de saída 1 não falharam na produção inicial de pseudocódigo; o erro registrado é `data.code=cleanup_incomplete`, em `close_binary`, após a observação parcial.
- Diagnóstico: o Ghidra 12.1.4 não conseguiu remover seus diretórios privados temporários porque arquivos de cache `db.1.gbf` estavam em uso (Win32 32). `stderr` estava vazio; o JSON de stdout contém `data.details.partial_observation`.
- `___tmainCRTStartup`, endereço `0x406b5d`: pseudocódigo extraído para a pasta local `diagnostico/cobertura_20261014/recuperacao_parcial_cleanup_incomplete_20261014`; 2.761 caracteres. É função de runtime Visual C++ 2010, não deve contar automaticamente como lógica própria do ConnectServer.
- `MSVCR100.DLL::type_info::_type_info_dtor_internal_method`, endereço `0x40726a`: pseudocódigo extraído para a mesma pasta; 306 caracteres. Contém aviso de recuperação de jumptable e salto indireto tratado como chamada; manter como resultado suspeito/não validado, não afirmar que a semântica do destrutor foi recuperada.
- As duas observações reportam o SHA-256 do executável ConnectServer.exe como `f25eecb6c65214b7658fc2a8a1b0c786bbb1b718a78405a8e4c61d88c9477aed`. Isso foi conferido nos metadados das observações; a verificação local independente posterior do arquivo continua recomendada.
- Os artefatos locais originais e a extração de recuperação não foram enviados ao GitHub; esta atualização registra somente a continuidade.
- Não repetir as duas análises antes de verificar estado dos processos/arquivos temporários do Ghidra. Não apagar manualmente diretórios temporários retidos. Próxima etapa: verificar integridade da recuperação e do executável; depois validar/normalizar as 57 saídas brutas aprovadas em pasta versionada nova, sem sobrescrever originais, e reconciliar cobertura contra inventário.

## 17. Normalização e reconciliação por endereço do lote ConnectServer (2026-10-14)

**Resultados confirmados pelo usuário no PowerShell local.** Os artefatos e relatórios detalhados continuam no computador local; este registro documenta somente os resultados resumidos.

### Normalização versionada
- Pasta criada sem sobrescrever os artefatos anteriores: `D:\\REA\\pacote_muserver\\ConnectServer\\diagnostico\\cobertura_20261014\\normalizacao_v1_20261014`.
- Foram criados 57 pseudocódigos `.c.txt` e 57 metadados `.metadata.json`, extraídos das 57 saídas brutas validadas.
- As duas falhas originais de limpeza do Ghidra foram preservadas separadamente; a normalização não as reclassificou como sucessos.
- A revisão semântica permanece pendente. Pseudocódigo extraído/normalizado não equivale a comportamento plenamente compreendido.
- O SHA-256 esperado do executável ConnectServer foi confirmado pelo script local como correspondente ao arquivo examinado.

### Inventário e reconciliação
- A estrutura correta do arquivo `diagnostico/inventario_procedimentos.json.txt` é um objeto cujo array de registros está em `raw_result`; cada registro usa `value` e `address`. Não procurar a lista em `normalized_result` no nível raiz.
- `raw_result`: 332 registros; 133 funções internas nomeadas `FUN_<hex>` e 140 importações externas. Os 59 símbolos do lote são nomes adicionais/nomes próprios, não necessariamente funções com prefixo `FUN_`.
- Diretório `funcoes`: 266 arquivos com padrão `FUN_<hex>.(c.txt|json.txt)`, correspondendo a 133 pares somente se cada pseudocódigo tiver seu metadado correspondente; uma checagem de pareamento por chave continua recomendada antes de afirmar integridade completa.
- A reconciliação por endereço entre `manifesto.csv` do lote `lote_simbolos_internos_20261014_183049` e `raw_result` encontrou os 59 de 59 endereços; zero ausentes. Amostra conferida: `Catch_All@0040277d` em `0x40277d`, `Catch_All@00402b83` em `0x402b83`, `Catch_All@00402c07` em `0x402c07`, `Catch_All@004044c6` em `0x4044c6` e `__security_check_cookie` em `0x4068c0`.
- A primeira tentativa de reconciliação que informou zero itens do inventário foi um erro de leitura do script e deve ser considerada supersedida por esta reconciliação por endereço. Não houve evidência de ausência dos 59 símbolos.
- A classificação dos símbolos não `FUN_` permanece heurística e requer revisão semântica; handlers, funções de runtime, auxiliares, thunks e símbolos de segurança não devem ser tratados automaticamente como lógica de negócio.

### Estado e próximos passos
- A reconciliação por endereço do lote de 59 está concluída; a revisão semântica e a auditoria de pareamento/contagem de cobertura do inventário não estão concluídas.
- Não reexecutar os 57 sucessos e não apagar nem sobrescrever os artefatos anteriores.
- Próximo passo imediato: executar uma verificação compacta de pareamento dos 133 nomes `FUN_<hex>` (um `.c.txt` e um `.json.txt` por nome) e resumir divergências, se houver. Depois, registrar o fechamento da etapa ConnectServer e iniciar DATA-01 com inventário somente de leitura dos diretórios `Data` do MuServer, sem alterar arquivos originais.

## 18. Fechamento da cobertura estrutural dos artefatos FUN_ de ConnectServer (2026-10-14)

Resultado confirmado pelo usuário via PowerShell local, com leitura apenas do diretório `D:\\REA\\pacote_muserver\\ConnectServer\\funcoes`:
- Arquivos correspondentes ao padrão auditado: 266.
- Nomes únicos `FUN_<hex>`: 133.
- Pares completos (exatamente um `.c.txt` e um `.json.txt` por nome): 133.
- Nomes sem pseudocódigo: 0.
- Nomes sem metadados: 0.
- Pares com arquivos duplicados: 0.

Isso encerra a verificação estrutural de pareamento dos 133 artefatos FUN_ e, junto da reconciliação anterior, confirma que os 59 endereços do lote foram encontrados no inventário. Não demonstra que toda função interna do executável foi descompilada, nem que os pseudocódigos foram semanticamente revisados. Os 57 resultados normalizados são símbolos do lote por endereço e não devem ser somados automaticamente aos 133 pares FUN_ como se fossem conjuntos disjuntos.

### Encerramento da etapa ConnectServer — escopo preciso
- Concluído: inventário de 332 registros interpretado corretamente; 133 pares FUN_ estruturalmente íntegros; 59/59 endereços do lote reconciliados; 57 saídas brutas validadas e normalizadas em diretório separado; duas observações de `cleanup_incomplete` preservadas e documentadas como casos especiais/suspeitos.
- Continua pendente: revisão semântica, triagem definitiva dos símbolos não FUN_, confirmação da qualidade de pseudocódigo de cada função e análise funcional integrada do ConnectServer.
- Preservação: não reexecutar sucessos, não sobrescrever artefatos anteriores, não apagar caches temporários do Ghidra manualmente e manter o binário original imutável.

### Próxima etapa autorizada: DATA-01 — inventário somente de leitura
Iniciar inventário dos diretórios chamados `Data` dentro de `D:\\Muserver-GitHub\\MuServer - Copia`, registrando caminhos, extensões, tamanhos e hashes em saída versionada nova sob `D:\\REA\\pacote_muserver\\DATA\\inventario`. Não alterar, renomear, mover ou apagar arquivos do MuServer. Primeiro localizar as pastas `Data` e contar arquivos; só então ampliar para hashing/inventário detalhado. Registrar a data da execução e a raiz examinada, e preservar qualquer relatório preexistente.

## 19. DATA-01 — descoberta inicial dos diretórios Data (2026-10-14)

Resultado fornecido pelo usuário após execução local de busca somente leitura sob `D:\\Muserver-GitHub\\MuServer - Copia`:
- Foram localizados 12 diretórios cujo nome é `Data` (a busca não diferencia maiúsculas/minúsculas).
- Contagens por diretório:
  - `Data`: 1.631 arquivos recursivos; 12.124.621 bytes.
  - `GameServer\Data`: 9; 110.898 bytes.
  - `GameServerCS\DATA`: 9; 110.857 bytes.
  - `MHPServer\Data`: 8; 311.087 bytes.
  - `SpeedServer\Data`: 1.236; 11.381.496 bytes.
  - `SpeedServer\GameServer\Data`: 9; 110.936 bytes.
  - `SubSalatest\Data`: 1.613; 11.834.457 bytes.
  - `SubSalatest\GameServer\Data`: 9; 110.937 bytes.
  - `Tools\Editores\MuEditor\Data`: 3; 1.212.780 bytes.
  - `Tools\Gerador de Main\MAIN_INFO v41 - Season 6\CheckSum\Data`: 36; 9.419.180 bytes.
  - `Tools\Gerador de Main\MAIN_INFO v41 - Season 6\Premium\Data`: 467; 53.716.669 bytes.
  - `Tools\Gerador de Main\MAIN_INFOS6.20\CheckSum\Data`: 29; 5.994.734 bytes.
- O usuário não incluiu o cabeçalho inicial do resumo nem o total de erros de acesso na saída enviada; esses dados permanecem não confirmados.
- A contagem recursiva inclui subdiretórios e, como diretórios Data podem estar aninhados ou conter cópias de arquivos semelhantes, as contagens não devem ser somadas para declarar quantidade de arquivos únicos do repositório.
- Próxima ação: criar um inventário detalhado versionado em uma pasta nova, contendo caminho relativo/absoluto, extensão, tamanho e SHA-256 de cada arquivo dentro de cada diretório Data, sem modificar arquivos de origem. Preservar relatórios preexistentes e incluir tratamento de erros explícito. Depois agrupar por extensão e hashes para localizar duplicatas exatas sem apagá-las.

## 20. DATA-02B — inventário detalhado com SHA-256 concluído (2026-10-14)

Resultado confirmado pelo usuário após execução local:
- Diretórios `Data` encontrados: 12.
- Arquivos inventariados com SHA-256: 5.059.
- Falhas reportadas no cálculo/coleta: 0.
- Saída: `D:\\REA\\pacote_muserver\\DATA\\inventario\\DATA02B_20261014_193103`.
- Inventário principal: `inventario_arquivos.csv`.
- Resumo por extensão: `resumo_por_extensao.csv`.
- Relatório de possíveis duplicatas exatas: `duplicatas_sha256.csv`.
- Tentativa anterior `DATA02_20261014_192844` preservada; inválida porque o ambiente não possui `[IO.Path]::GetRelativePath()`. A falha foi corrigida por cálculo de caminho relativo baseado em prefixo.
- Dez extensões mais frequentes: `.txt` 4.038; `.ozt` 313; `.att` 276; `.lua` 122; `.dat` 117; `.ozj` 106; `.bmd` 56; `.rar` 10; `.smd` 8; `.db` 5.
- O total de 5.059 representa entradas encontradas recursivamente em cada diretório Data, não necessariamente 5.059 arquivos únicos em todo o repositório, pois pastas aninhadas e cópias podem se sobrepor.
- Próxima etapa: examinar o resumo completo por extensão e quantificar os grupos de SHA-256 repetidos; em seguida classificar os formatos e identificar arquivos de dados relevantes para a lógica do servidor. Não apagar, mover ou sobrescrever arquivos de origem.

## 21. DATA-03 — distribuição por extensão e duplicação (2026-10-14)

Resultado fornecido pelo usuário a partir de `DATA02B_20261014_193103/inventario_arquivos.csv`:
- Entradas no inventário: 5.059.
- Hashes SHA-256 distintos: 2.227.
- Grupos de hashes repetidos: 1.574.
- Entradas pertencentes a grupos repetidos: 4.406.
- Extensões por entradas / hashes distintos: `.txt` 4.038 / 1.630; `.ozt` 313 / 239; `.att` 276 / 82; `.lua` 122 / 65; `.dat` 117 / 60; `.ozj` 106 / 100; `.bmd` 56 / 30; `.rar` 10 / 6; `.smd` 8 / 6; `.db` 5 / 3; `.tga` 4 / 4; `.hit` 3 / 1; `.jpg` 1 / 1.
- Exemplos de duplicação exata observada: `003 - Harold the Wandering Merchant.txt` em múltiplas cópias Data; `Kalimdor.OZT` repetido 42 vezes; `Rank05.bmd` repetido 19 vezes; arquivos de terreno `Terrain44.att` e `Terrain40.att` repetidos entre cópias Data.
- A tabela de bytes por extensão retornou zero por um erro na expressão `Measure-Object`; os totais de bytes por extensão não estão validados e precisam ser recalculados com `Measure-Object -Property Bytes -Sum`.
- As extensões indicam prioridades iniciais: `.txt` e `.lua` para inspeção estática de configuração/lógica; `.att`, `.dat`, `.bmd`, `.hit` para identificação estrutural; recursos gráficos devem ser classificados separadamente. Não inferir semântica apenas pela extensão.
- Próximo passo: recalcular corretamente bytes por extensão e produzir uma amostra de arquivos de configuração relevantes por caminho (por exemplo `Data\\Command.txt`, `Data\\ConnectMember.txt`, `Data\\Character\\DefaultClassInfo.txt`, `Data\\Custom\\CustomMonster.txt`, `Data\\Custom\\CustomMove.txt`, `Data\\Custom\\CustomMix.txt`) para então ler conteúdo em modo somente leitura. Não alterar ou apagar originais.


## 22. DATA-04B — agregação de bytes e arquivos prioritários (2026-10-14)

Resultado confirmado pelo usuário após execução local de leitura do inventário `DATA02B_20261014_193103`:

- Inventário: 5.059 entradas; 0 campos `Bytes` inválidos; total agregado de 106.438.652 bytes.
- Soma de bytes por extensão, entradas / hashes distintos / bytes:
  - `.txt`: 4.038 / 1.630 / 16.185.560 bytes.
  - `.ozt`: 313 / 239 / 29.602.275 bytes.
  - `.att`: 276 / 82 / 18.088.764 bytes.
  - `.lua`: 122 / 65 / 712.705 bytes.
  - `.dat`: 117 / 60 / 844.743 bytes.
  - `.ozj`: 106 / 100 / 10.460.504 bytes.
  - `.bmd`: 56 / 30 / 22.720.859 bytes.
  - `.rar`: 10 / 6 / 1.378.962 bytes.
  - `.smd`: 8 / 6 / 2.723.532 bytes.
  - `.db`: 5 / 3 / 245.960 bytes.
  - `.tga`: 4 / 4 / 3.408.022 bytes.
  - `.hit`: 3 / 1 / 47.628 bytes.
  - `.jpg`: 1 / 1 / 19.138 bytes.
- Os nove nomes-alvo foram encontrados no inventário. O resultado contém várias cópias sob `Data`, `SpeedServer\Data` e `SubSalatest\Data`; portanto, comparar por nome não basta para presumir equivalência funcional.
- Diferenças relevantes observadas:
  - `Command.txt`: três cópias de 17.350 bytes com SHA-256 diferentes.
  - `ConnectMember.txt`: duas cópias idênticas de 207 bytes e uma cópia diferente de 291 bytes.
  - `CustomEventDrop.txt`: cópia de 2.724 bytes diferente das duas cópias idênticas de 4.021 bytes.
  - `CustomMix.txt`: três cópias distintas, de 640.238, 476.645 e 633.517 bytes.
  - `CustomMonster.txt`: uma cópia de 29.043 bytes e duas cópias idênticas de 29.064 bytes.
  - `CustomMapRequirement.txt`, `CustomMove.txt`, `DefaultClassInfo.txt` e `DefaultClassStatus.txt` são idênticos entre as três cópias listadas.
- As contagens de TXT/LUA por pasta confirmadas no passo anterior foram: `Data` 1.509; `SubSalatest\Data` 1.491; `SpeedServer\Data` 1.114; e pastas de ferramentas com 18, 15 e 13 entradas, respectivamente. São contagens por raiz de inventário e não necessariamente arquivos únicos globais.
- O total agregado é a soma das entradas inventariadas, incluindo cópias repetidas; não deve ser descrito como tamanho de arquivos únicos.
- Limite de escopo: isso confirma integridade aritmética dos tamanhos e localiza divergências por cópia, mas não determina qual configuração está ativa em runtime nem qual cópia deve ser considerada canônica. Não executar nem modificar o servidor nesta fase.

### Próxima etapa DATA-05
1. Fazer leitura estática, sem edição, das nove configurações-alvo nas cópias encontradas.
2. Comparar conteúdo e estrutura das cópias divergentes, começando por `ConnectMember.txt`, `Command.txt`, `CustomEventDrop.txt`, `CustomMonster.txt` e `CustomMix.txt`.
3. Registrar diferenças por caminho e linhas, sem substituir arquivos, eleger uma cópia canônica sem evidência ou expor segredos/licenças.
4. Depois decidir quais arquivos adicionais de `.txt`/`.lua` merecem inspeção; manter a auditoria do GameServer para depois da etapa Data, conforme a sequência definida.


## 23. Ajuste de escopo DATA-05 — somente a pasta Data principal (2026-10-14)

Decisão explícita do usuário:
- A análise funcional de configurações/scripts será limitada à pasta principal `D:\\Muserver-GitHub\\MuServer - Copia\\Data`.
- Não comparar nem analisar cópias em `SpeedServer\\Data` ou `SubSalatest\\Data`; essas cópias não fazem parte do escopo prioritário atual.
- O inventário DATA02B e os resultados globais anteriores permanecem como evidência histórica de descoberta e integridade, mas divergências entre essas cópias secundárias não serão investigadas nem usadas para decidir a configuração canônica.
- DATA-05 prossegue com leitura estática, somente leitura, dos arquivos-alvo exclusivamente na pasta principal `Data`. Preservar os arquivos originais; não executar o servidor nem editar configurações nesta etapa.


## 24. DATA-05 — resumo da pasta Data principal e ConnectMember (2026-10-14)

Resultados fornecidos pelo usuário após comandos locais de inventário/leitura, limitados a `D:\\Muserver-GitHub\\MuServer - Copia\\Data`:

- Resumo da pasta principal: 1.631 arquivos, 12.124.621 bytes; 1.481 arquivos `.txt` e 28 arquivos `.lua`.
- `ConnectMember.txt`: 207 bytes; SHA-256 `4EFF5FF9C7144883CB4212E1C72DC1F85DE1DB1C61D56AFF6DED628D828F2AB5`.
- Conteúdo observado: comentário `Account Allowed to enter server`, quatro identificadores de conta (`pvp01`, `pvp02`, `pvp03`, `pvp04`) e terminador `end`.
- Interpretação provisória: o comentário indica uma lista de contas permitidas para entrada. A leitura estática, isoladamente, não prova qual executável consome o arquivo nem se ele participa da autenticação principal.
- `Command.txt`: 17.350 bytes; SHA-256 `526F26427A36899AC0DA1FED935DEDB9669AFEE6BD8EFE79D320B2CCDC32AC5F`. O cabeçalho descreve uma tabela de comandos com parâmetros de habilitação, dinheiro, níveis, resets, atraso, nível de Game Master e moedas (WC/WP/GP), e referencia `Command.dat` como configuração complementar. A primeira entrada visível é `/move`; a linha exibe `Enable = 1` e descrição de movimento para um mapa. É necessário conferir o restante da tabela e a estrutura dos campos antes de concluir regras por nível/permissão.
- Escopo permanece somente a pasta principal `Data`; não comparar/analisar `SpeedServer\\Data` ou `SubSalatest\\Data`. Somente leitura; não executar nem editar o servidor/configurações nesta fase.

### Próximo passo
Continuar a inspeção de `Command.txt` a partir de amostras controladas das entradas e localizar `Command.dat` apenas dentro da pasta principal `Data`. Não compartilhar conteúdo sensível caso apareça.


## 25. DATA-05 — estrutura inicial de Command.txt (2026-10-14)

Resultados fornecidos pelo usuário a partir de leitura local somente leitura do arquivo principal `Data\\Command.txt`:

- Metadados previamente confirmados: 17.350 bytes; SHA-256 `526F26427A36899AC0DA1FED935DEDB9669AFEE6BD8EFE79D320B2CCDC32AC5F`.
- A expressão de amostragem identificou linhas numeradas com nome de comando entre aspas. As primeiras 15 entradas têm índices 0–14 e incluem `/move`, `/post`, `/f`, `/a`, `/v`, `/e`, `/c`, `/pk`, `/zen`, `/evo`, `/bau`, `/reset`, `/gmmove`, `/!` e `/track`.
- O cabeçalho descreve campos por categoria `AL0`–`AL3`, dinheiro, níveis, resets, delay, GameMaster e moedas. A amostra mostra diferenças de valores entre comandos; para `/evo` e `/reset`, os quatro valores de habilitação exibidos são zero; para `/zen`, esses quatro valores também são zero, embora outros campos da linha sejam distintos. Isso é uma leitura dos valores da configuração, não confirmação de comportamento em execução.
- A busca recursiva por nome exato `Command.dat` dentro da pasta principal `Data` não encontrou arquivo correspondente. Isso não determina se o arquivo está ausente em todo o pacote, se é procurado fora de `Data`, ou se o comentário do TXT está desatualizado.
- Próxima verificação: contar as entradas identificadas, resumir nomes e valores de habilitação sem imprimir o arquivo completo, e procurar referências textuais a `Command.dat` somente dentro da pasta principal `Data`. Não modificar configuração nem executar binários.


## 26. DATA-05 — catálogo de comandos e busca por Command.dat (2026-10-14)

Resultado fornecido pelo usuário a partir da leitura estática de `Data\\Command.txt` e busca textual em arquivos `.txt`/`.lua` dentro da pasta principal `Data`:

- A expressão usada para reconhecer linhas numeradas com nome entre aspas retornou 103 linhas, com índices aparentes de 0 a 102.
- A saída colada pelo usuário contém repetição aparente do índice 95 e do comando `/setbuff`; isso deve ser validado diretamente no arquivo antes de afirmar que a duplicação existe no conteúdo original.
- A lista inclui comandos de movimentação, chat, atributos, reset, administração, economia, eventos, grupos/parties, lojas offline e outras funções, conforme nomes observados. Essa classificação é indicativa pelos nomes, não confirmação de comportamento interno.
- Busca por `Command.dat` nos arquivos `.txt`/`.lua` da pasta principal `Data` retornou somente o comentário da linha 19 de `Command.txt`: `// Obs: Others configurations see Command.dat`. Não foram encontradas outras referências textuais no conjunto pesquisado. Isso não prova que o arquivo inexista fora da pasta Data ou que não seja acessado de modo indireto.
- Próximo passo: verificar diretamente no arquivo os índices/nomes duplicados e gerar um resumo compacto de campos de habilitação para cada comando, sem imprimir a tabela inteira. Continuar em modo somente leitura, sem executar o servidor e sem analisar cópias secundárias.


## 27. DATA-05 — duplicata confirmada em Command.txt (2026-10-14)

Resultado da extração local fornecida pelo usuário a partir do arquivo principal `Data\\Command.txt`:

- Linhas reconhecidas pela expressão regular: 104.
- Índices distintos: 103.
- Nomes distintos: 103.
- Duplicata confirmada na própria extração do arquivo: índice `95`, nome `/setbuff`, aparece duas vezes.
- Classificação: anomalia estrutural confirmada na configuração. O efeito em runtime é desconhecido; não presumir se o parser ignora, substitui ou trata de outro modo a entrada repetida.
- Não editar o arquivo para remover a duplicata nesta fase. Próximo passo: inspecionar as linhas físicas correspondentes e resumir os campos iniciais de habilitação com posições ainda tratadas como provisórias, sem inferir comportamento além do cabeçalho.


## 28. DATA-05 — duplicata idêntica e amostra de campos iniciais (2026-10-14)

Resultado fornecido pelo usuário a partir da inspeção local de `Data\\Command.txt`:

- As duas ocorrências de índice 95 / `/setbuff` estão nas linhas físicas 120 e 121. As linhas apresentadas têm os mesmos valores numéricos e descrição; a diferença visível é de espaçamento. Tratar como duplicata estrutural provavelmente redundante, mas o comportamento do parser para índices repetidos permanece desconhecido. Não remover nem alterar.
- A extração dos quatro primeiros números após o nome mostra padrões `1,1,1,1`, `0,0,0,0`, `2,1,1,1` e `0,1,1,1` em entradas diferentes. O cabeçalho rotula os campos de habilitação por categorias `AL0`–`AL3`; não inferir ainda o efeito operacional do valor 2 ou o comportamento em runtime.
- Exemplos visíveis: `/move` e `/post` começam com `1,1,1,1`; `/evo`, `/reset` e `/mreset` com `0,0,0,0`; `/make` e `/setbuff` com `2,1,1,1`; `/maketarget` com `0,1,1,1`.
- Próximo passo: calcular distribuição desses quatro valores e localizar os demais arquivos de configuração de comandos somente dentro da pasta principal `Data`, sem analisar cópias secundárias ou editar arquivos.


## 29. DATA-05 — distribuição de flags e arquivos complementares de comandos (2026-10-14)

Resultado fornecido pelo usuário após leitura somente leitura de `Data\\Command.txt` e enumeração de nomes contendo `Command` exclusivamente sob a pasta principal `Data`:

- Distribuição dos quatro primeiros campos numéricos após índice/nome, interpretados provisoriamente como flags AL0–AL3 conforme o cabeçalho: `0,0,0,0`: 13 entradas; `0,1,1,1`: 1; `1,0,0,0`: 1; `1,1,1,1`: 40; `2,1,1,1`: 49. Total 104 entradas reconhecidas, incluindo a duplicata de `/setbuff`.
- Não concluir ainda o significado operacional do valor 2; o campo separado `GameMaster` também está presente no cabeçalho e precisa ser analisado separadamente.
- Arquivos com nome contendo `Command` encontrados na pasta principal Data: `Command.txt` (17.350 bytes), `Custom\\CustomCommandDescription.txt` (1.633 bytes) e `Custom\\CustomNpcCommand.txt` (863 bytes). A busca anterior por `Command.dat` encontrou apenas a menção no comentário de `Command.txt`; nenhum arquivo exato `Command.dat` foi localizado dentro da Data principal.
- Próximo passo: ler os cabeçalhos e amostras limitadas de `CustomCommandDescription.txt` e `CustomNpcCommand.txt`, registrando seus hashes, sem modificar arquivos nem analisar cópias secundárias.


## 30. DATA-05 — primeiros dados dos arquivos complementares de comandos (2026-10-14)

Saída local fornecida pelo usuário, escopo restrito à pasta principal `D:\\Muserver-GitHub\\MuServer - Copia\\Data`:

- `Custom\\CustomCommandDescription.txt`: 1.633 bytes; SHA-256 `F66060E41079A3C893620E2C41A81C2FB0B05EB667BB5A66BD3796F548173188`. Cabeçalho declara colunas `ID`, `Command`, `String`; texto de comando com tamanho máximo de 127 caracteres. A observação do arquivo diz que o comando não pode existir em `Command.txt` nem em `CustomMove.txt`. A amostra inicial termina antes de mostrar registros de configuração, portanto os registros ainda não foram inspecionados.
- `Custom\\CustomNpcCommand.txt`: 863 bytes; SHA-256 `2DFB8B211D028C6616BF572D0165FE95D27064D03A2096D292D08846FDBD1F2C`. Cabeçalho declara `Index, Npc, NpcMap, NpcX, NpcY, NpcTalk, Command Text`. Registros visíveis: índice 0, NPC 601, mapa 00, coordenadas 143/145, NpcTalk 1, comando `/quest`; índice 1, NPC 793, mapa 92, coordenadas 188/102, NpcTalk 1, comando `/control` (comentário Tower Arka War Event); índice 2, NPC 789, mapa 00, coordenadas 121/133, NpcTalk 1, comando `/zen 20`; índice 3, NPC 785, mapa 00, coordenadas 143/145, NpcTalk 1, comando `/enterarena` (comentário Zen). Há `end` após os registros. Essas linhas sugerem comandos associados a NPCs, mas não confirmam a semântica dos campos nem o comportamento em runtime.
- Próximo passo: ler o conteúdo completo dos dois arquivos pequenos, mantendo os hashes e verificando os registros reais antes de cruzar nomes com `Command.txt` e `CustomMove.txt`. Não modificar arquivos.


## 31. DATA-05 — conteúdo completo de descrições customizadas e comandos de NPC (2026-10-14)

O usuário forneceu a saída integral dos dois arquivos; hashes confirmados iguais aos registrados na seção 30.

- `Custom\\CustomCommandDescription.txt` tem 24 linhas. A única linha de exemplo de configuração é comentada: `// 0 "/testando" "Comando teste"`. Portanto, no conteúdo visível não há registro ativo de descrição personalizada; isso não prova se o sistema aceita arquivo vazio ou se há comportamento adicional no executável. O cabeçalho avisa que o texto tem tamanho máximo de 127 caracteres e que o comando não deve existir em `Command.txt` nem em `CustomMove.txt`.
- `Custom\\CustomNpcCommand.txt` contém quatro registros ativos e um terminador `end`: (índice 0) NPC 601, mapa 00, X=143, Y=145, NpcTalk=1, texto `/quest`; (1) NPC 793, mapa 92, X=188, Y=102, NpcTalk=1, `/control`, comentário Tower Arka War Event; (2) NPC 789, mapa 00, X=121, Y=133, NpcTalk=1, `/zen 20`, comentário Zen; (3) NPC 785, mapa 00, X=143, Y=145, NpcTalk=1, `/enterarena`, comentário Zen. Interpretação limitada à estrutura declarada; ainda não sabemos como o servidor resolve ou executa esses textos.
- Próxima verificação estática: pesquisar `/quest`, `/control`, `/zen`, `/enterarena` e `/testando` em `Command.txt` e `CustomMove.txt` dentro da Data principal, sem percorrer cópias secundárias; também registrar tamanho/hash de `CustomMove.txt` e mostrar apenas linhas correspondentes e contexto imediato.
