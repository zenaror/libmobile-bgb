# Orientações para agentes

Este projeto liga a biblioteca libmobile ao emulador BGB (programa `mobile`). A linha ativa é a branch `feature/full_server`.

## Antes de começar

1. Consulte a memória interna e depois a OMM no escopo `libmobile-bgb`. Leia `project-base` com `get_role` na retomada; consulte global quando ajudar. Memória não substitui fatos confirmados no Git, código e evidências.
2. Para REON, libmobile ou o protocolo Mobile Adapter GB, use a skill `reon-libmobile-expert` da OMM.
3. Confira branch, HEAD, alterações locais e revisão de `subprojects/libmobile`, que é mantido separadamente. Preserve seu gitlink e o outro checkout. O remote deste projeto é apenas `https://github.com/zenaror/libmobile-bgb`.
4. Use o [mapa do README](README.md#project-map) para código, documentação, ferramentas e continuidade. `source/` e `test.py` na raiz são os caminhos válidos deste projeto; não reorganize o upstream por estética.

## Durante o trabalho

- Compile em `build/` (Linux) e em `build-windows/` com `../configure --host=x86_64-w64-mingw32 --without-system-libmobile` (Windows).
- Rode os testes com `ln -sf build/mobile mobile && python3 test.py`. Os testes de device-auth exigem UID efetivo 0 no ambiente de teste; veja limites no [README](README.md#integration-and-validation).
- Não faça `git commit` nem `git push` sem autorização do Operador para o trabalho. Reescrita de histórico e force-push precisam de autorização específica na própria sessão.
- Não insira automaticamente trailers `Co-authored-by` de Claude/Anthropic nos commits. Preserve autores, coautores e créditos humanos legítimos.
- Não crie opção para trocar o servidor de device-auth.
- Não grave senhas, chaves (como a `device_auth_key`) nem dados pessoais em arquivos, commits ou na OMM.

## Ao terminar

Registre na OMM o conhecimento duradouro novo, com a origem (arquivo, commit ou conversa), e deixe um handoff com estado, bloqueios e próximos passos. Mantenha a memória interna e a OMM atualizadas.

## Cleanup e retenção

A política canônica local está no [README.md — Cleanup and retention](README.md#cleanup-and-retention). Aplicar no fechamento/checkpoint/pausa: limpar somente saídas comprovadamente pertencentes à tarefa, após preservar evidências e verificar processos/caminhos; preservar dados reais, releases e recuperação Git. Não usar limpeza ampla nem tratar ignorados/não rastreados como descartáveis.
