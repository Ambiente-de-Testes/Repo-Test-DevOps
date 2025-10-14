Esse repositório contem algumas workflows personalizadas para equipes sem o plano Enterpise para permitir criar permissões personalizadas

Primeiramente vamos pegar alguns dados, escolha um repositório que vai ficar responsavel por guardar os scripts e executar as actions
<img width="1918" height="103" alt="image" src="https://github.com/user-attachments/assets/d8028418-4276-4fca-a769-f5f0420ade8c" />

Aqui no meu caso eu tenho uma organização chamada Ambiente-de-Testes, e dentro dela um repositório chamado Repo-Test-DevOps

Tendo definido aonde você irá guardar os scripts vamos acessar a aba Actions no repositório
<img width="1918" height="880" alt="image" src="https://github.com/user-attachments/assets/df79a7c4-6b64-427a-94b6-0394b7b7f3ed" />
E escolher a opção Simple Workflow clicando em Configure

Será aberta essa tela 
<img width="1915" height="727" alt="image" src="https://github.com/user-attachments/assets/196bf6cd-43ef-4fd8-b7a8-62fae83cdfad" />
Por hora podemos remover todo o código que já vem escrito

E aonde o nome atualmente é blank.yml vamos renomear para status.yml

E esse será o novo conteudo que você irá utilizar no campo Code: 

```
name: Get Project Info

on:
  workflow_dispatch:

jobs:
  get-info:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.UPDATE_TOKEN }}
          script: |
            const projectId = "PVT_kwDODixzhM4BFb_a";
            
            const query = `
              query ($projectId: ID!) {
                node(id: $projectId) {
                  ... on ProjectV2 {
                    title
                    fields(first: 20) {
                      nodes {
                        ... on ProjectV2SingleSelectField {
                          id
                          name
                          options {
                            id
                            name
                          }
                        }
                      }
                    }
                  }
                }
              }
            `;
            
            const result = await github.graphql(query, { projectId });
            
            console.log("Informações do Projeto:");
            console.log(JSON.stringify(result, null, 2));
            
            // Procura pelo campo de status
            const statusField = result.node.fields.nodes.find(f => 
              f.id === "PVTSSF_lADODixzhM4BFb_azg2xh6E"
            );
            
            if (statusField) {
              console.log("\n Status disponíveis:");
              statusField.options.forEach(opt => {
                console.log(`   ${opt.name}: ${opt.id}`);
              });
            }
```

Com o código já pronto podemos clicar em Commit Changes, sera cliado uma pasta nesse repósitório chamada workflows e dentro dela você poderá ver o arquivo e editar se desejar, más agora vamos precisar acionar uma ação

<img width="1906" height="864" alt="image" src="https://github.com/user-attachments/assets/468d586f-ba95-40b6-8eb1-d29c352f260f" />

De volta na tela de Actions selecione o Get Project Info, o meu eu coloquei (Temporario) depois do nome más o seu não deve ter esse texto adicional

<img width="1917" height="874" alt="image" src="https://github.com/user-attachments/assets/fc857480-dfa4-46d0-a9e6-7a0b8d546961" />

Clicando nele podemos ver que nada foi executado ainda poís esse é uma action de ativação manual, então vamos clicar em Run workflow para disparar a ação

<img width="1903" height="871" alt="image" src="https://github.com/user-attachments/assets/6962add0-ca2b-4735-bb18-3f21ff483fbd" />

Ao recerregar a tela você deve ver que a ação foi iniciada

<img width="1914" height="871" alt="image" src="https://github.com/user-attachments/assets/2e2c4ca3-2029-4a05-9426-6d29f15f9e82" />

Clicando nela você será levado para uma nova página que mostra alguns detalhes da execução da ação

<img width="1918" height="724" alt="image" src="https://github.com/user-attachments/assets/4cfadbf2-7148-4087-b09e-aad7173b36dc" />

Clique aonde está sendo indicado para visualizar para como foi a execução
Se tudo funcionar corretamente você verá essa página com os passos da execução

<img width="1917" height="718" alt="image" src="https://github.com/user-attachments/assets/f1af5821-f16d-473a-94aa-9ddf07be1955" />

Clicando em Run action veremos o resultado da nossa consulta
<img width="1321" height="673" alt="image" src="https://github.com/user-attachments/assets/6e1258fc-cfaa-4908-a7bb-fe41ebba555d" />

Copie todo o conteudo da busca e salve em um arquivo a parte no seu computador, iremos usar essas informações posteriormente

--------------------------------------------------------------------------------------------------------------------------------

Agora iremos configurar o primeiro Workflow de produção para o projeto

Este workflow foi projetado para otimizar o gerenciamento de issues em projetos, facilitando a colaboração entre equipes com diferentes níveis de acesso ao repositório de código.
Funcionamento

    Criação de Issues: Membros da equipe sem acesso direto ao repositório de código podem criar issues em um repositório padrão onde têm permissões

    Replicação Automática: As issues são automaticamente replicadas para o repositório de código destino

    Sistema de Labels: A replicação ocorre com base nas labels configuradas previamente no workflow

Benefícios

    Permite que analistas de casos participem do fluxo de trabalho sem necessidade de acesso completo ao código

    Mantém a sincronização entre repositórios

    Automatiza o processo de tracking de issues

    Preserva a segurança do repositório de código principal

Fluxo do Processo

    Criação da issue no repositório padrão

    Aplicação das labels apropriadas

    Replicação automática para o repositório de código

    Acompanhamento e gestão centralizada

Este workflow garante que todas as partes interessadas possam contribuir eficientemente com o projeto, independentemente das permissões de acesso ao código-fonte.

Vamos usar uma organização como exemplo, defini como nome para ela "Ambiente-de-Testes"
<img width="1918" height="880" alt="image" src="https://github.com/user-attachments/assets/e4c62d0b-84a5-4a7a-82c1-653d363a8c5a" />

Primeiramente vamos até as configurações do ser perfil para gerar uma chave de acesso para utilizar nas Actions 
<img width="380" height="874" alt="image" src="https://github.com/user-attachments/assets/7a9ef8d5-a1e7-4938-bacf-b8eb746e79ae" />

No final dessa página o item que queremos acessa está em Developer settings
<img width="1914" height="880" alt="image" src="https://github.com/user-attachments/assets/42e570f1-eb0e-4ba0-be46-ec8f6e294b91" />

Em primeiro momento iremos criar um Fine-grainer token, posteriormente iremos criar um clássico também para demosntrar o funcionamento
<img width="1912" height="784" alt="image" src="https://github.com/user-attachments/assets/cb15b594-3e9d-44fc-9744-92398516bc75" />

Clicando em Generate new token
<img width="1911" height="772" alt="image" src="https://github.com/user-attachments/assets/9acf14bc-353a-4e48-b68f-8b4a86054d86" />

Vamos preencher as informações básicas 
<img width="1912" height="883" alt="image" src="https://github.com/user-attachments/assets/e5f933fd-ed9f-42c1-9e03-94d7195841c7" />
E precisamos nos atentar no Resource owner
Nesse caso como vamos trabalhar com uma organização iremos mudar o dono do meu perfil para a organização que vamos utilizar

<img width="1206" height="679" alt="image" src="https://github.com/user-attachments/assets/c686510a-592e-48c6-8156-2e2a3a1d826e" />

E vamos definir como No expiration para evitar a manutenção por hora

<img width="1918" height="862" alt="image" src="https://github.com/user-attachments/assets/8fe6f3ed-d720-46fb-a0a1-30f3bacb52e8" />

Os repositórios utilizar apenas alguns selecionados, más fique a vontade caso queri fazer de outra maneira
<img width="1918" height="876" alt="image" src="https://github.com/user-attachments/assets/39da62d9-cb87-437e-9ce2-76ecc895a7a5" />

E em Permissions vamos pesquisar a opção Issues e marcar como habilitada
<img width="1918" height="880" alt="image" src="https://github.com/user-attachments/assets/e7680b8c-fb3f-49a7-a76f-7c6efcd6fbcd" />

Definiremos o Access como 'Read and Write' e o campo Metadata já deve vir automaticamente como 'Read-only'
<img width="1918" height="880" alt="image" src="https://github.com/user-attachments/assets/697688ac-8146-401d-9267-51231537cb1e" />

Após isso poderemos clicar no botão logo abaixo 'Generate token'

<img width="1890" height="871" alt="image" src="https://github.com/user-attachments/assets/6b57f9f1-d826-4629-9412-37461c0820b5" />

Agora é a hora de copiar esse token e salvar em um bloco de notas, poís não iremos conseguir ver novamente por aqui 
<img width="1915" height="865" alt="image" src="https://github.com/user-attachments/assets/5db9a245-1703-4552-ae6e-5abd04f86458" />

Tendo esse token salvo podemos ir nas configurações da organização aonde vamos precisar ajustar alguns campos
<img width="1918" height="868" alt="image" src="https://github.com/user-attachments/assets/f3dd880d-c827-441a-be12-f19d6de8c119" />
<img width="1906" height="872" alt="image" src="https://github.com/user-attachments/assets/e1ed3dd7-9ea1-406a-9e39-338a774f46bc" />

E com tudo configurado podemos clicar em Save
<img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/9bdf6407-cdf3-46ea-abbf-d0f9453e9079" />

Agora vamos voltar no repositório usamos para criar o primeiro workflow

<img width="1919" height="794" alt="image" src="https://github.com/user-attachments/assets/54f06194-1b81-4bd1-bb9c-7023e731dc37" />

E vamos clicar em Settings
<img width="1919" height="248" alt="image" src="https://github.com/user-attachments/assets/b92b8b6d-9257-4cf9-97a3-74d55406d2dc" />

Queremos encontrar a opção Secrets and variables e vamos clicar em Actions
<img width="1919" height="872" alt="image" src="https://github.com/user-attachments/assets/ae73f665-6746-4c23-b35e-11d2d8ebe1b5" />

Aqui iremos criar um New repository secret
<img width="1918" height="875" alt="image" src="https://github.com/user-attachments/assets/b39b052b-9178-4ac8-a22f-be8f73bb529e" />

Defina um nome de fácil memorização e no campo abaixo iremos colocar aquela key que criamos anteriormente e deixamos reservada
<img width="1919" height="878" alt="image" src="https://github.com/user-attachments/assets/6879f01f-35e9-4589-b104-dea8537115d8" />

Podemos clicar em Add Secret para confirmar e o nosso secret poderá ser encontrado na página em que estavamos antes

<img width="1919" height="717" alt="image" src="https://github.com/user-attachments/assets/eb11ddd5-5c76-4751-9d50-d6a2933b980a" />

Com tudo isso pronto, agora podemos fazer partir para o workflow de fato

<img width="1919" height="304" alt="image" src="https://github.com/user-attachments/assets/ff4a8ad7-c23f-4e27-83f8-1a1918a03c48" />
<img width="1919" height="725" alt="image" src="https://github.com/user-attachments/assets/7c282b8d-d6e2-4bf1-a849-f0352f96acf7" />


E novamente vamos limpar o texto que vem preenchido e renomear o nome do arquivo como recreate-issues.yml
Deixei um script pré configurado e vamos precisar editar as informações contidas nele

```
name: Sync Issues to Target Repo

on:
  issues:
    types: [labeled]

jobs:
  sync:
    runs-on: ubuntu-latest
    if: github.event.action == 'opened' || github.event.action == 'labeled'

    steps:
      - name: Extract data
        id: extract
        run: |
          echo "title<<EOF" >> $GITHUB_OUTPUT
          echo "${{ github.event.issue.title }}" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
          
          echo "body<<EOF" >> $GITHUB_OUTPUT
          echo "${{ github.event.issue.body }}" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT

      - name: Determine target repo from label
        id: target
        run: |
          echo "Detectando label..."
          LABEL_NAME="${{ github.event.label.name }}"
          
          if [ -z "$LABEL_NAME" ]; then
            echo "Nenhuma label no evento direto, tentando pegar a primeira label da issue..."
            LABEL_NAME=$(echo "${{ github.event.issue.labels[0].name }}" | tr -d '[:space:]')
          fi

          if [ -z "$LABEL_NAME" ]; then
            echo "Nenhuma label encontrada. Encerrando."
            exit 1
          fi

          echo "repo=$LABEL_NAME" >> $GITHUB_OUTPUT
          echo "Repositório alvo: $LABEL_NAME"

      - name: Create issue in target repository
        run: |
          echo "Criando issue em Ambiente-de-Testes/${{ steps.target.outputs.repo }}..."
          RESPONSE=$(curl -s -o response.json -w "%{http_code}" -X POST \
            -H "Authorization: token ${{ secrets.GH_TOKEN }}" \
            -H "Accept: application/vnd.github.v3+json" \
            https://api.github.com/repos/Ambiente-de-Testes/${{ steps.target.outputs.repo }}/issues \
            -d @- <<EOF
          {
            "title": "${{ steps.extract.outputs.title }}",
            "body": "${{ steps.extract.outputs.body }}\n\n---\n_Origin: https://github.com/${{ github.repository }}/issues/${{ github.event.issue.number }}_"
          }
          EOF
          )

          echo "Código de resposta: $RESPONSE"
          cat response.json
          if [ "$RESPONSE" -ne 201 ]; then
            echo "Falha ao criar a issue."
            exit 1
          fi
```
Esse é o texto que você pode colar no lugar e os campos destacados são aqueles que você vai precisar alterar agora, sendo o nome do projeto e o nome do token
Se você sequiu a risca a documentação o nome do seu token vai ser 'DEVOPS_TOKEN'
<img width="1919" height="759" alt="image" src="https://github.com/user-attachments/assets/0904784b-176f-4d64-b5c4-d673e937a01b" />

Feito isso podemos fazer o commit direto
<img width="1919" height="866" alt="image" src="https://github.com/user-attachments/assets/34d085a2-7675-4379-9b48-18c5280557f4" />

E estará pronta esperando em Actions
<img width="1919" height="814" alt="image" src="https://github.com/user-attachments/assets/806a35d6-8b13-4de7-9442-483a92607e14" />

Essa executa altomaticamente conforme o a ação que definimos no código, para essa situação a ação que eu defini foi a labeled, ou seja, quando usa issue receber uma label a Action sera ativada

Na tela do projeto iremos adicionar uma nova issue para testar

<img width="1919" height="880" alt="image" src="https://github.com/user-attachments/assets/4874f233-38df-471a-9c11-01cc1ebd83be" />
<img width="1919" height="879" alt="image" src="https://github.com/user-attachments/assets/71afa5ef-6f62-4b4a-95ba-f0304a71e327" />
<img width="1919" height="877" alt="image" src="https://github.com/user-attachments/assets/4942de9c-53cd-44c5-b1ed-e1940e6b717e" />

Para funcionar como queremos você vai precisar ter uma label como o mesmo nome de um de seus repositórios/destino, como na imagem dentre as labels padrão do GitHub eu tenho uma com o mesmo nome do repositório que eu vou usar para teste
<img width="1919" height="876" alt="image" src="https://github.com/user-attachments/assets/b24e74c5-9765-444b-bc27-13943bae1770" />

Logo após selecionar a label a Action foi chamada
<img width="1919" height="876" alt="image" src="https://github.com/user-attachments/assets/8c258781-b5bf-4d83-97d0-79b19d4aa7be" />

E podemos ver o resultado como esperado
<img width="1919" height="873" alt="image" src="https://github.com/user-attachments/assets/6b31b01f-06d6-44e8-ac19-a3cd14857a83" />










































