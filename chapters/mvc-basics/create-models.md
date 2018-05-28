## Criar models
Existem duas classes model que precisam ser criadas: um model que represente um item da to-do list armazenado no banco de dados (as vezes chamado de **entity**), e o model que será associado à view (o **MV** no MVC) e enviado ao navegador do usuário. Porque ambos podem ser referidos como "models", eu irei me referir ao último como um **view model**.

Primeiro, crie uma classe chamada `TodoItem` no diretório Models:

**Models/TodoItem.cs**

```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace AspNetCoreTodo.Models
{
    public class TodoItem
    {
        public Guid Id { get; set; }
        
        public bool IsDone { get; set; }

        [Required]
        public string Title { get; set; }

        public DateTimeOffset? DueAt { get; set; }
    }
}
```

Esta classe define o que o banco de dados precisará armazenar para cada item da to-do list: um ID, um title ou nome, se o item está completo, e qual é a data de vencimento. Cada linha define uma propriedade da classe:

* A propriedade **Id** é um guid, ou um **g**lobally **u**nique **id**entifier (identificador único global). Guids (ou GUIDs) são longas strings de letras e números, como `43ec09f2-7f70-4f4b-9559-65011d5781bb`. Porque guids são aleatórios e é extremamente improvável de serem acidentalmente duplicados, eles são comumente utilizados como IDs únicos. Você também pode usar um número (inteiro) como um ID de entidade no banco de dados, mas você precisaria configurar seu banco de dados para sempre incrementar o número quando novas linhas forem adicionadas ao banco. Guids são geradas aleatoriamente, então você não precisa se preocupar com auto-incremento.

* A propriedade **IsDone** é um boolean (valores true/false). Por padrão, será `false` para todos os novos itens. Mais tarde você escreverá código para mudar essa propriedade para `true` quando o usuário clicar no checkbox de um item na view.

* A propriedade **Title** é uma string (texto). Ela irá guardar o nome ou a descrição do item da to-do list. O atributo `[Required]` diz ao ASP.NET Core que essa string não pode ser nula ou vazia.

* A propriedade **DueAt** é um `DateTimeOffset`, que é um tipo do C# que armazena um date/time stamp juntamente com um fuso horário UTC. Armazenar a data, hora, e fuso horário juntos facilita processar datas precisamente em sistemas com diferentes fusos horários.

Observou o ponto de interrogação `?` depois do tipo `DateTimeOffset`? Isso marca a propriedade DueAt como **nullable**, ou opcional. Se a `?` não fosse incluída, todo item da to-do list precisaria ter uma data de vencimento. As propriedades `Id` e `IsDone` não são marcadas como nullable, então elas são obrigatórias e sempre terão um valor (ou um valor padrão).

> Strings no C# são sempre nullable, então não há necessidade de marcar a propriedade Title como nullable. No C# strings podem ser nulas, vazias, ou conterem texto.

Cada propriedade é seguida por `get; set;`, que é uma forma abreviada de informar que a propriedade é de leitura/escrita (ou, mais tecnicamente, possui os métodos getter e setter).

Neste ponto, não importa qual tecnologia de banco de dados será utilizada. Pode ser SQL Server, MySQL, MongoDB, Redis, ou algo mais exótico. Este model define como a linha ou entrada do banco de dados irá parecer no C# então você não precisa se preocupar com afazeres de baixo nível de banco de dados no seu código. Este estilo simples de model as vezes é chamado de "plain old C# object" ou POCO.

### O view model

Frequentemente, o model (entidade) que você armazena na base de dados é similar mas não *exatamente* igual ao model que você quer usar no MVC (o view model). Neste caso, o model `TodoItem` representa um único item no banco de dados, mas na view pode precisar exibir dois, dez, ou uma centena de itens (dependendo de quanto o usuário está procrastinando).

Por este motivo, o view model precisa ser uma classe separada que contém um array de `TodoItem`s:

**Models/TodoViewModel.cs**

```csharp
namespace AspNetCoreTodo.Models
{
    public class TodoViewModel
    {
        public TodoItem[] Items { get; set; }
    }
}
```

Agora que você tem alguns models, é hora de criar uma view que irá pegar um `TodoViewModel` e renderizar o HTML correto para exibir ao usuário sua to-do list.
