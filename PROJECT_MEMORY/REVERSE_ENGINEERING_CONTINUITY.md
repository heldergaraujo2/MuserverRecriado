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

## 8. Lote-piloto ConnectServer — estado ainda NÃO confirmado

Foi enviado ao usuário um script PowerShell para decompilar até 10 funções pendentes de ConnectServer, em sequência. O script:
- lê `procedimentos_internos.csv`;
- seleciona nomes que correspondem a `^FUN_[0-9A-Fa-f]+$`;
- tenta pular nomes que já têm `funcoes\<nome>.json.txt`;
- chama REA com `--procedure <endereço> --provider ghidra --format json --filter-output normalized_result`;
- grava JSON, pseudocódigo C, logs por função, `resultado_lote_<timestamp>.csv` e `RESUMO_LOTE_<timestamp>.md`.

**A saída do lote-piloto ainda não foi recebida/confirmada neste registro.** Pode estar em execução, ter terminado ou ter falhado. Não iniciar outro REA/Ghidra antes de o usuário informar o resultado e/ou de verificar o estado local.

Pontos a validar no resultado:
1. O resumo indica quantas funções foram selecionadas, puladas, concluídas e falharam?
2. Cada JSON é JSON válido e contém o formato esperado?
3. O pseudocódigo `.c.txt` existe e é não vazio quando a saída permitir extração?
4. Cada função tem log e status por separado?
5. O script não marcou como sucesso uma saída vazia, erro de limpeza ou pseudocódigo parcial?
6. O script pulou resultados antigos apenas pela existência do JSON, sem validá-los? Se sim, reconciliar isso em vez de assumir que estão íntegros.
7. Quantas funções restam das 192 entradas não externas? A lista não deve se limitar a nomes `FUN_...`; entradas `Catch_All` e outros símbolos devem ser classificados separadamente.
8. Atualizar este arquivo com a saída real do usuário e o próximo passo específico.

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

**Aguardar/verificar o lote-piloto de ConnectServer que foi enviado, sem iniciar outro decompilador em paralelo.** O usuário deve compartilhar o arquivo `RESUMO_LOTE_<timestamp>.md` criado pelo script e, se houver falha, os trechos relevantes de `resultado_lote_<timestamp>.csv` e os logs correspondentes.

Quando houver saída:
- auditar o lote;
- corrigir a lógica de validação/retomada se necessário;
- medir cobertura contra as entradas reais do inventário;
- depois criar/rodar um lote sequencial retomável para o restante do ConnectServer, antes de avançar ao próximo binário conforme a fila global.

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

Comece lendo este arquivo integralmente. Pergunte pelo resumo do lote-piloto apenas se ele ainda não estiver disponível no contexto. Antes de propor qualquer comando, verifique se o lote está ativo e se os artefatos citados existem. Continue do item **12 — Próxima etapa exata**. Um comando por vez, aguarde a saída, mantenha análise estática, preserve tudo, cubra o MuServer inteiro e atualize este documento no GitHub depois de cada resultado confirmado.
