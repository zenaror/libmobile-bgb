# Orientações para agentes

Este projeto liga a biblioteca libmobile ao emulador BGB (programa `mobile`). A linha ativa é a branch `feature/full_server`.

## Antes de começar

1. Consulte a memória interna do seu agente sobre este projeto. Depois consulte a OMM no escopo `libmobile-bgb` e, quando ajudar, no `global`. Essa é a ordem de consulta, não de autoridade: confirme os fatos no código, no Git e nos testes.
2. Para REON, libmobile ou o protocolo Mobile Adapter GB, use a skill `reon-libmobile-expert` da OMM.
3. Confira a branch, o commit, as alterações locais e a revisão de `subprojects/libmobile`, que tem histórico próprio. Preserve o trabalho que não é seu.

## Durante o trabalho

- Compile em `build/` (Linux) e em `build-windows/` com `../configure --host=x86_64-w64-mingw32 --without-system-libmobile` (Windows).
- Rode os testes com `ln -sf build/mobile mobile && python3 test.py`. Os testes de device-auth precisam de root.
- Não faça `git commit` nem `git push` sem a palavra do Rafael. Reescrita de histórico e force-push precisam da palavra dele na própria sessão.
- Não crie opção para trocar o servidor de device-auth.
- Não grave senhas, chaves (como a `device_auth_key`) nem dados pessoais em arquivos, commits ou na OMM.

## Ao terminar

Registre na OMM o conhecimento duradouro novo, com a origem (arquivo, commit ou conversa), e deixe um handoff com estado, bloqueios e próximos passos. Mantenha a memória interna e a OMM atualizadas.
