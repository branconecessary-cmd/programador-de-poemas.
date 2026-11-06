# Autenticação, autorização e gestão de credenciais

> Documentação técnica pública  
> Repositório: `branconecessary-cmd/programador-de-poemas.`  
> Escopo: acesso ao repositório GitHub e limites da autenticação no projeto.

## 1. Escopo e constatação técnica

O **Programador de Poemas** é apresentado neste repositório como uma metodologia autoral de colaboração entre pessoas e inteligência artificial. A árvore pública examinada contém documentação, manifesto, apresentação e estudos de caso. Ela não contém uma aplicação web ou serviço de API com implementação própria de login, cadastro, sessão, OAuth, emissão de tokens ou validação de credenciais.

Por isso, é importante separar dois assuntos:

1. **Autenticação da plataforma GitHub:** controla quem pode consultar ou modificar este repositório.
2. **Autenticação interna do Programador de Poemas:** não há, na estrutura pública examinada, um mecanismo próprio de autenticação de usuários a documentar.

Esta página descreve o primeiro mecanismo e registra explicitamente a ausência do segundo. Não atribui ao projeto componentes que não estão implementados no código público.

## 2. Componentes envolvidos

| Componente | Responsabilidade |
|---|---|
| GitHub | Hospeda o repositório, aplica as regras de acesso e registra operações. |
| Conta GitHub | Identifica a pessoa ou organização que solicita acesso. |
| Permissões do repositório | Definem se a operação é pública, de leitura ou de escrita. |
| HTTPS + token | Permite operações Git autenticadas por HTTPS, quando configurado. |
| SSH + chave | Permite autenticar operações Git por chave pública/privada. |
| Git Credential Manager | Pode armazenar e fornecer credenciais de forma protegida ao cliente Git. |
| API do GitHub | Recebe chamadas autenticadas e aplica as permissões associadas à credencial. |

Esses componentes pertencem à infraestrutura GitHub e às ferramentas de desenvolvimento. Não são módulos de autenticação implementados pelo Programador de Poemas.

## 3. Fluxo de requisição

### 3.1 Consulta pública

Como o repositório tem visibilidade pública, uma pessoa pode abrir páginas, ler arquivos e consultar o histórico sem autenticar uma conta GitHub.

```text
Navegador
   |
   | HTTPS - solicitação de leitura
   v
GitHub
   |
   | verifica a visibilidade pública
   v
Repositório
   |
   v
README, documentação, arquivos e histórico
```

### 3.2 Operação de escrita via Git

Para enviar alterações, o GitHub exige uma identidade autenticada e permissão de escrita. O método concreto depende da configuração local.

**HTTPS:** o cliente Git solicita autenticação. Um token compatível pode ser fornecido por um gerenciador de credenciais. O GitHub valida a credencial e verifica as permissões antes de aceitar o push.

```text
Git local
   |
   | git push via HTTPS
   v
Gerenciador de credenciais
   |
   | fornece credencial armazenada
   v
GitHub
   |
   | valida identidade e permissões
   v
Aceita ou rejeita a operação
```

**SSH:** o cliente demonstra posse da chave privada correspondente à chave pública cadastrada na conta ou organização. O GitHub verifica a chave e as permissões aplicáveis.

```text
Git local
   |
   | conexão SSH + prova criptográfica
   v
GitHub
   |
   | valida chave pública cadastrada
   | verifica acesso ao repositório
   v
Aceita ou rejeita a operação
```

### 3.3 Operações pela API

Integrações e aplicações que utilizam a API do GitHub enviam requisições HTTPS. Quando a rota exige autenticação, a chamada apresenta uma credencial compatível, normalmente no cabeçalho de autorização. O GitHub valida a credencial, verifica escopos ou permissões e decide se a operação é permitida.

Exemplo ilustrativo de chamada, usando uma variável de ambiente já configurada no ambiente local:

```bash
curl --request GET \
  --url "https://api.github.com/repos/branconecessary-cmd/programador-de-poemas." \
  --header "Accept: application/vnd.github+json" \
  --header "Authorization: Bearer $GITHUB_TOKEN"
```

O exemplo demonstra o padrão de uma chamada autenticada à API. **Não significa que o repositório implemente ou utilize internamente esse endpoint para autenticar usuários.** Para leitura pública, o cabeçalho de autorização não é necessário.

## 4. Como credenciais e tokens são tratados

O repositório não contém código próprio para receber, emitir, armazenar, renovar ou revogar tokens de usuários. O ciclo de vida da credencial é responsabilidade do GitHub e da ferramenta ou integração utilizada para acessar a plataforma.

Boas práticas aplicáveis a quem mantém ou integra este repositório:

- Não inserir tokens, senhas, chaves privadas ou arquivos de credenciais no código, README, issues ou commits.
- Preferir tokens de curta duração ou com expiração, quando disponíveis.
- Conceder somente as permissões necessárias e limitar o acesso aos repositórios pertinentes.
- Utilizar armazenamento seguro de credenciais, como o Git Credential Manager ou o cofre de segredos do ambiente.
- Para automações, armazenar segredos em mecanismos próprios de secrets, nunca em texto puro no repositório.
- Revogar imediatamente credenciais expostas e revisar o histórico quando houver vazamento.
- Não compartilhar tokens entre pessoas ou serviços sem necessidade e controle.
- Revisar periodicamente as chaves SSH, aplicações autorizadas e permissões concedidas.

### Exemplo seguro de configuração local

O token não deve ser escrito diretamente no comando, no arquivo versionado ou no código-fonte. Uma aplicação pode receber o valor por variável de ambiente:

```bash
export GITHUB_TOKEN="valor-fornecido-fora-do-repositorio"
```

O valor acima é apenas um marcador explicativo. Não é uma credencial real e não deve ser substituído por um segredo em documentação pública.

## 5. OAuth, aplicações e integrações

Uma integração conectada à conta GitHub pode utilizar OAuth ou outro mecanismo de autorização disponibilizado pela plataforma. Nesse cenário, o fluxo de consentimento e a emissão ou gestão dos tokens pertencem ao GitHub e à aplicação integradora.

O repositório do Programador de Poemas não apresenta, em seus arquivos públicos examinados, um servidor OAuth, callback de autenticação, client secret, banco de sessões ou middleware de autorização. Portanto, não há base para afirmar que o projeto implementa OAuth.

Quando uma ferramenta externa recebe autorização para operar no GitHub, essa autorização não deve ser confundida com uma funcionalidade de autenticação do projeto. A integração atua com as permissões concedidas à conexão e às políticas da plataforma.

## 6. Limites e segurança

A visibilidade pública permite leitura sem autenticação, mas não concede permissão de escrita. A alteração de arquivos continua condicionada à autenticação e às permissões reconhecidas pelo GitHub.

Esta documentação não afirma que o projeto possui:

- login ou cadastro próprio;
- autenticação multifator implementada pela aplicação;
- gerenciamento interno de sessão;
- emissão, renovação ou revogação própria de tokens;
- controle de acesso baseado em papéis dentro de uma aplicação;
- armazenamento de senhas ou credenciais de usuários.

Esses recursos somente poderão ser documentados como componentes do projeto se uma implementação correspondente for adicionada ao repositório e puder ser verificada.

## 7. Referências oficiais

- [GitHub Docs - About authentication to GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-authentication-to-github)
- [GitHub Docs - Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub Docs - Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub Docs - About authentication with SAML single sign-on](https://docs.github.com/en/authentication/authenticating-with-single-sign-on/about-authentication-with-single-sign-on)
- [GitHub REST API - Authenticating to the REST API](https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api)

## 8. Nota de transparência

Esta documentação descreve a autenticação da plataforma que hospeda o projeto e diferencia esse mecanismo de uma autenticação que seria implementada pela própria aplicação. A análise foi limitada aos arquivos e à estrutura pública disponíveis no repositório no momento da documentação.

**Responsável pelo projeto:** Patrick Branco Campos  
**Projeto:** Programador de Poemas  
**Repositório:** https://github.com/branconecessary-cmd/programador-de-poemas.

© Patrick Branco Campos. Todos os direitos reservados.
