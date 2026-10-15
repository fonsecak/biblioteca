# Trabalho 5 — Pipeline CI com GitHub Actions

**Disciplina:** Cloud Computing e DevOps Avançado  
**Curso:** Sistemas para Internet | 6º Período  
**Professor:** Diogo Ranghetti  

---

## Integrantes do Grupo

- **Gabriel Fonseca Proença** - RA: 14608
- **Guilherme Carrera** - RA: 14651
- **Kaiky Massaki Ishibashi** - RA: 14535
- **Marcos Kaiky Rodrigues Garcia** - RA: 14610


## Objetivo

Configurar um pipeline de integração contínua reproduzível para o projeto Biblioteca, usando GitHub Actions, Maven Wrapper e Java 21.

## Repositório e workflow

- Repositório: https://github.com/fonsecak/biblioteca
- Workflow: `.github/workflows/ci.yml`
- Gatilhos: push e pull request para `master` e `main`, além de execução manual (`workflow_dispatch`).

### Pré-requisitos (Execução Local)
* **Java JDK:** Versão 21
* **Git:** Para clonagem e controle de versão

## Etapas do pipeline

1. Faz checkout do commit.
2. Configura Java 21 Temurin e cache das dependências Maven.
3. Executa `./mvnw --batch-mode --no-transfer-progress clean verify`.
4. Se build e testes passarem, prepara e publica o JAR executável como artefato.

Falhas em compilação ou testes interrompem a execução antes da criação do artefato. O job tem permissão de leitura do conteúdo do repositório.

## Artefato e rastreabilidade

O artefato publicado contém:

- JAR executável da aplicação;
- `BUILD_INFO.txt`, com commit, branch, ID da execução, tentativa, horário UTC e versão Java;
- `SHA256SUMS.txt`, com o checksum SHA-256 do JAR.

O artefato fica associado à execução do workflow e tem retenção configurada para 30 dias (sujeita ao limite definido no repositório).

## Instruções de Execução

### Execução Automática (via CI)
1. Realize uma alteração no código-fonte num ramo de desenvolvimento ou diretamente na branch configurada.
2. Faça o commit das alterações:

   ```
   git add .
   git commit -m "feat: descrição da alteração"
   ```

3. Envie as alterações para o repositório remoto (`git push origin main`) ou abra um **Pull Request** apontando para a branch `main`.
4. Vá para o separador **[Actions / CI-CD]** do repositório para acompanhar a execução automática em tempo real.

### Como reproduzir localmente

Requisitos: Java 21 e acesso ao Maven Central para o Maven Wrapper baixar a distribuição e dependências.

Linux/macOS:

```bash
export API_SECURITY_TOKEN_SECRET="<segredo-local-com-pelo-menos-32-bytes>"
./mvnw --batch-mode --no-transfer-progress clean verify
```

Windows PowerShell:

```powershell
$env:API_SECURITY_TOKEN_SECRET = "<segredo-local-com-pelo-menos-32-bytes>"
.\mvnw.cmd --batch-mode --no-transfer-progress clean verify
```

O segredo JWT é lido da variável `API_SECURITY_TOKEN_SECRET`; não deve ser commitado nem impresso em logs. O workflow usa um valor descartável somente durante os testes, sem depender de segredos cadastrados no GitHub.

## Evidências das execuções

- **Sucesso:** [execução CI #38659555550](https://github.com/fonsecak/biblioteca/actions/runs/38659555550), commit `a6c97d402a0cb5e816a44a4451039cc104c9743e`. Build e testes passaram e o artefato foi publicado.
- **Falha detectada:** [execução CI #38659432166](https://github.com/fonsecak/biblioteca/actions/runs/38659432166). Um teste foi configurado para falhar intencionalmente na branch temporária `evidence/ci-failure`; a etapa de testes falhou e as etapas de artefato foram puladas. A branch foi removida após a execução; o registro e os logs permanecem no Actions.

## Investigar uma falha

Na aba **Actions**, abra a execução, expanda o primeiro passo vermelho e examine o log do Maven. O Maven indica testes, erros de compilação e mensagens relevantes. Corrija o problema, rode `clean verify` localmente e envie um novo commit. Em falha de build ou testes, o artefato não é gerado.
