# Core.ts
```typescript
const jsonFilePath = __dirname + '/data.temp.json'; // Cria o caminho do arquivo JSON
const list: string[] = await loadFromFile(); // Cria uma lista de strings e preenche com os dados do arquivo usando a função loadFromFile().

// função responsável por ler o arquivo
async function loadFromFile() {
  try {
    const file = Bun.file(jsonFilePath); // acessa o caminho jsonFilePath
    const content = await file.text(); // ler o arquivo como texto
    return JSON.parse(content) as string[]; // transforma o JSON em um array de strings
  } 
  // identifica um erro
  catch (error: any) { 
    if (error.code === 'ENOENT')
      return []; // se o erro não for encontrado vai retornar um array vazio
    throw error; // não sei
  }
}

// função assincrona que salva os dados no arquivo
async function saveToFile() {
  try {
    await Bun.write(jsonFilePath, JSON.stringify(list)); // converte a lista para JSON
  } 
  // identifica um erro
  catch (error: any) {
   throw new Error("Erro ao salvar os dados no arquivo: " + error.message); // lança um erro personalizado
  }
}

// Função que adiciona um novo item

async function addItem(item: string) {
  list.push(item); // adiciona um item no final da lista
  await saveToFile(); // salva a lista
}

// Função que pega todos os itens
async function getItems() {
  return list;
}

// função que atualiza um item
async function updateItem(index: number, newItem: string) {
  if (index < 0 || index >= list.length) // identifica se o index é inválido
    throw new Error("Index fora dos limites"); // lança um erro se o index não existir
  list[index] = newItem; // atualiza o index para a posição informada
  await saveToFile(); // salva a lista
}

// função para remover um item
async function removeItem(index: number) {
  if (index < 0 || index >= list.length) // identifica se o index é invalido
    throw new Error("Index fora dos limites"); // lança um erro se ele não existir
  list.splice(index, 1); //remove o item
  await saveToFile(); // salva a lista
}


export default { addItem, getItems, updateItem, removeItem }; // exporta essas funções
```
---
# api.turma02.ts

```typescript
import todo from "./core.ts";// importa as funções
// servidor http
const server = Bun.serve({
  port: 3000,

  routes: {
    "/": new Response(Bun.file("./public/index.html")), //  retorna a rota principal

    "/api/todo": { // Rota TODO

      GET: async () => { // Método GET para listar os itens
        const items = await todo.getItems() // Obtém todos os itens
        return Response.json(items) // Retorna os itens em formato JSON
      },

      POST: async (req) => { // Método POST para adicionar os itens
        const data = await req.json() as any; // lê os arquivos
        const item = data.item || null;
        if (!item) // verifica se o item foi enviado
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 }); //retorna o erro 400(não encontrado)
        await todo.addItem(item); // adciona os itens
        return Response.json(data); // retorna os dados
      },
    },

    "/api/todo/:index": {
      PUT: async (req) => {// Método PUT para atualizar os itens
        const index = parseInt(req.params.index); // converte o index para número
        if (isNaN(index)) // verifica se o índice é inválido
          return Response.json('Índice inválido. um número inteiro é esperado.', { status: 400 }); // retorna um erro 400
        const data = await req.json() as any; // lê os dados
        const newItem = data.newItem || null; // obtém um novo valor
        if (!newItem) // verifica se foi enviado
          return Response.json('Por favor, forneça um novo item para atualizar.', { status: 400 });// retorna um erro 400
        try {
          await todo.updateItem(index, newItem);// atualiza o item
          return Response.json(`Item no índice ${index} atualizado para "${newItem}".`); // retorna mesnsagem de sucesso
        } catch (error: any) {
          return Response.json(error.message, { status: 400 }); //retorna o erro
        }
      },

      DELETE: async (req) => {// Método DELETE para deletar os itens
        const index = parseInt(req.params.index);// converte index para número
        if (isNaN(index))
          return Response.json('Índice inválido.', { status: 400 }); // se o inde for errado retorna um erro 400
        try {
          await todo.removeItem(index); // remove o nome da lista
          return Response.json(`Item no índice ${index} removido com sucesso.`);  // retorna sucesso
        } catch (error: any) { //procura um erro
          return Response.json(error.message, { status: 400 }); // retorna um erro 400
        }
      },
    },

    // EXEMPLO BÁSICO

    "/api/exemplo": {
      GET: () => {
        return new Response(`Esse é o exemplo: ${Date.now()}`)
      },

      POST: async (req) => {
        const data = await req.json() as any;
        data.recebidoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },
    },

    "/api/exemplo/:id": {
      PUT: async (req, params) => {
        const { id } = req.params;
        const data = await req.json() as any;
        data.id = id;
        data.recebidoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },

      PATCH: async (req, params) => {
        const { id } = req.params;
        const data = await req.json() as any;
        data.chavesAtualizadas = Object.keys(data);
        data.id = id;
        data.atualizadoEm = new Date().toLocaleDateString("pt-BR");
        return Response.json(data);
      },

      DELETE: (req, params) => {
        const { id } = req.params;
        return new Response(`Recurso com id ${id} deletado`, { status: 200 });
      }
    }
    // FIM DO EXEMPLO BÁSICO
  },

  async fetch(req) {
    return new Response(`Not Found`, { status: 404 });
  },
});

console.log(`Server running at http://localhost:${server.port}`);
```
