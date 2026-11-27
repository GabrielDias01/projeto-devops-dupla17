# Projeto DevOps: dupla17

Site: https://gabrieldias01.github.io/projeto-devops-dupla17/ (status em /status.html)
Imagem: ghcr.io/gabrieldias01/projeto-devops-dupla17

## O fluxo completo
1. O `git push` envia o commit para a `main`, e a tag `v*` é enviada separadamente com `git push origin vX`.
2. A cada push na `main`, o fluxo CI roda os testes do pytest, e o fluxo do site roda os testes de novo antes de publicar o `status.html` no GitHub Pages.
3. Quando a tag `v*` chega, o fluxo "Publicar imagem" testa o código, entra no ghcr.io com o token do GitHub e faz o build com `APP_VERSION` igual à tag.
4. A imagem é publicada no ghcr.io com o nome da tag (v1, v2, v3).
5. No terminal, `docker pull` baixa a imagem, `docker run` executa com a porta 5007 mapeada e o `curl` em `/saude` devolve a versão.

## Linha do tempo
- v1: primeira imagem publicada, com APP_VERSION=v1. /saude devolve "versao":"v1".
- v2: mudança da porta de 5007 para 5008 dentro de app/app.py (commit 95424eb). A imagem sobe, mas o mapeamento 5007 não responde: o curl falha e o docker logs mostra a API em 5008.
- v3: rollback no repositório com git revert do commit 95424eb (commit 7516a29). /saude volta a devolver "versao":"v3", e o status.html mostra o commit 7516a29.

## O que os testes não pegaram
Os testes usam o cliente do Flask, que chama a aplicação sem abrir a porta de rede. Por isso, a porta errada passou em todos os testes, e a v2 foi publicada sem erro no pipeline. Para pegar isso, faltaria um teste que suba o contêiner, faça um curl na porta publicada e falhe se não responder, antes de publicar a imagem.

## Quem fez o quê
**Gabriel Dias (GabrielDias01, aluno A):** Nos encontros anteriores, criei o `ci.yml` e o teste `test_GABRIEL.py`. Neste encontro, substituí o `pages.yml` pela versão que roda os testes antes de publicar e gera o `status.html` com o commit, a hora e o resultado dos testes, e confirmei que, se um teste falhar, o site anterior continua no ar. Depois que o Nickolas publicou a v1, conferi a visibilidade da imagem no ghcr.io: ela já estava pública, porque herdou a visibilidade do repositório, então não precisei mudar nada e anotei que minha previsão de `denied` não se confirmou. Rodei a v1, a v2 e a v3 no meu Codespace com `docker run` na porta 5007, vi o `curl` falhar na v2 e li no `docker logs` que a API estava na porta 5008. Fiz o rollback operacional voltando para a v1 sem mexer no código e cronometrei o tempo. Em seguida, fiz o rollback no repositório com `git revert` do commit 95424eb, o que criou o commit 7516a29 sem apagar o histórico, e publiquei a tag v3. Por fim, criei este README e revisei o conteúdo.

**Nickolas Pagagnelli (pagagnelli, aluno B):** A completar pelo aluno B