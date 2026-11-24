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

## Regras do Firestore (colar em Firestore → Regras, trocando SEU_EMAIL)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    function logado() { return request.auth != null && request.auth.token.email_verified == true; }
    function dono()   { return logado() && request.auth.token.email == 'SEU_EMAIL'; }
    function cad()    { return /databases/$(db)/documents/editores/$(request.auth.token.email); }
    function editor() { return dono() || (logado() && exists(cad())); }
    function admin()  { return dono() || (logado() && exists(cad()) && get(cad()).data.papel == 'Administrador'); }

    match /conteudo/{doc}   { allow read: if true;     allow write: if editor(); }
    match /editores/{email} { allow read: if editor(); allow write: if admin(); }
  }
}
```

- SEU_EMAIL (minúsculo) é o dono: entra mesmo antes de existir a lista de editores. As regras ficam só no console, não no repositório.
- `email_verified` impede que alguém crie uma conta com o e-mail de outra pessoa e entre: só vale quem abriu o link enviado ao e-mail.
- A lista de editores fica fora de "conteudo" para os e-mails não ficarem públicos.

## Primeiro acesso de cada pessoa

1. Um administrador adiciona a pessoa em **Acessos** e salva.
2. A pessoa abre o Estúdio, digita o e-mail e clica em **Primeiro acesso ou esqueci a senha**.
3. Recebe o link, cria a senha e entra. (Vale também para o dono, uma vez, para o e-mail ficar verificado.)

## Hospedagem

Pasta `/estudio/` do repositório `okrs-sharepoint` → `https://pdedbvo.github.io/okrs-sharepoint/estudio/`.
O código é público, mas não tem segredo: a chave do Firebase web é pública por natureza e quem protege os dados são as regras acima + login.

## Próximos passos

1. portal-dbvo e okrs-sharepoint passam a ler o banco (com reserva no código), testados em demo e com prints antes de publicar.
2. Momentos: um registro por foto, se precisar de mais de ~3.
