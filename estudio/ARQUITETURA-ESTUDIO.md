# Estúdio · site de configurações do SharePoint DBVO

Estado em 24/11/2026: publicado em https://pdedbvo.github.io/okrs-sharepoint/estudio/ (projeto Firebase **sharepoint-studio**). Teste sem mexer no banco: adicionar `?demo` ao link.

## Como as peças se ligam

```
Estúdio (login e-mail/senha)  ──grava──►  Firestore do projeto **sharepoint-studio**
                                               coleção "conteudo" (leitura pública, escrita só editores)
                                               coleção "editores" (só editores leem)
portal-dbvo  ──lê──►  conteudo/portal, conteudo/momentos          (dados.js vira reserva)
okrs-sharepoint ──lê──►  conteudo/empresas, times, empreendimentos, inside   (CONFIG vira reserva)
```

- Nenhum iframe muda. As páginas passam a ler o banco e, se ele falhar, usam o conteúdo que já está no código.
- Um documento por assunto: `portal`, `momentos`, `empresas`, `times`, `empreendimentos`, `inside`, no formato `{ v: <conteúdo>, em: <data>, por: <e-mail> }`.
- Fotos: reduzidas no navegador (1280 px, JPEG 70%, até ~300 KB cada) e gravadas no próprio documento. Sem Firebase Storage, sem plano pago. Limite do Firestore: 1 MB por documento, então "momentos" vira um documento por foto quando for preciso mais que ~3 fotos.
- Leitura pelas páginas: 1 leitura por documento por visita (com cache de 24 h no navegador, como o painel). Fica dentro da cota gratuita.

## Regras do Firestore (colar em Firestore → Regras)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    function logado() { return request.auth != null && request.auth.token.email != null; }
    function dono()   { return logado() && request.auth.token.email == 'SEU_EMAIL'; }
    function cad()    { return /databases/$(db)/documents/editores/$(request.auth.token.email); }
    function editor() { return dono() || (logado() && exists(cad())); }
    function admin()  { return dono() || (logado() && exists(cad()) && get(cad()).data.papel == 'Administrador'); }

    match /conteudo/{doc}   { allow read: if true;     allow write: if editor(); }
    match /editores/{email} { allow read: if editor(); allow write: if admin(); }
  }
}
```

- SEU_EMAIL (minúsculo) é o dono. As regras ficam só no console, não no repositório.
- **Cadastro pelo site desligado:** em Authentication → Configurações → Ações do usuário, desmarcar "Ativar criação (inscrição)". Assim só existe conta criada no console, e ninguém cria conta com o e-mail de outra pessoa. Por isso as regras não dependem de e-mail verificado (o link por e-mail falhou no Microsoft 365: o Safe Links abre e expira o link antes da pessoa).
- A lista de editores fica fora de "conteudo" para os e-mails não ficarem públicos.

## Liberar uma pessoa

1. No Estúdio, aba **Acessos**: adicionar nome, e-mail e papel, e salvar.
2. No Firebase, Authentication → Usuários → Adicionar usuário: mesmo e-mail e uma senha. Passar a senha para a pessoa.
3. "Esqueci a senha" no Estúdio manda link por e-mail, mas pode ser barrado pelo Microsoft 365. Alternativa: apagar e recriar o usuário no console com senha nova.

## Hospedagem

Pasta `/estudio/` do repositório `okrs-sharepoint` → `https://pdedbvo.github.io/okrs-sharepoint/estudio/`.
O código é público, mas não tem segredo: a chave do Firebase web é pública por natureza e quem protege os dados são as regras acima + login.

## Próximos passos

1. portal-dbvo e okrs-sharepoint passam a ler o banco (com reserva no código), testados em demo e com prints antes de publicar.
2. Momentos: um registro por foto, se precisar de mais de ~3.
