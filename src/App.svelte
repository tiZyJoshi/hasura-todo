<script>
  import client from "./graphql-client";
  import { gql } from "@apollo/client";

  let todo = {
    title: "",
  };
  async function insertTodo() {
    const mutation = gql`
      mutation InsertTodo($title: String!) {
        insert_todo(objects: { title: $title, completed: false }) {
          returning {
            id
          }
          affected_rows
        }
      }
    `;
    client.mutate({
      mutation: mutation,
      variables: {
        title: todo.title,
      },
    });
  }

  async function updateTodo(todo) {
    const mutation = gql`
      mutation UpdateCompleted($id: Int!, $completed: Boolean!) {
        update_todo(
          where: { id: { _eq: $id } }
          _set: { completed: $completed }
        ) {
          affected_rows
        }
      }
    `;
    client.mutate({
      mutation: mutation,
      variables: {
        id: todo.id,
        completed: todo.completed,
      },
    });
  }

  const query = gql`
    subscription TodosSubscription {
      todo(order_by: { id: desc }) {
        completed
        id
        title
      }
    }
  `;
  const todos = client.subscribe({
    query: query,
  });
</script>

<main>
  <form on:submit|preventDefault={insertTodo}>
    <input placeholder="enter TODO title" bind:value={todo.title} />
    <button type="submit">Submit</button>
  </form>
</main>

{#if $todos && $todos.data}
  <table>
    <thead>
      <tr>
        <td>ID</td>
        <td>Title</td>
        <td>Status</td>
      </tr>
    </thead>
    <tbody>
      {#each $todos.data.todo as t}
        <tr>
          <td>{t.id}</td>
          <td>{t.title}</td>
          <td
            ><label
              ><input
                type="checkbox"
                bind:checked={t.completed}
                on:change={() => updateTodo(t)}
              />{t.completed ? "done" : "pending"}</label
            ></td
          >
        </tr>
      {/each}
    </tbody>
  </table>
{/if}
