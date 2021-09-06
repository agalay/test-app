<template>
  <div class="container">
    <header>
      <h1>Posts</h1>

    </header>
   <main>
     <ul>
       <li v-for="{ id, title, description } in posts" :key="id">
         <h4>{{ title }}</h4>
         <p>{{ description  }}</p>
         <div class="btn-box">
           <button @click="remove(id)" style="margin-bottom: 5px" class="btn">Delete</button>
           <button @click="edit(id)" class="btn">Edit</button>
         </div>
       </li>
     </ul>
   </main>
    <footer>
      <form action="" @submit.prevent="updateOrCreate">
        <input type="text" placeholder="Title" name="title" v-model="data.title">
        <textarea cols="30" rows="10" placeholder="Description" name="description" v-model="data.description"></textarea>
        <button class="btn">{{ action }}</button>
      </form>
    </footer>
  </div>
</template>

<script>

export default {
  name: 'App',

  data() {
    return {
      posts: [],
      action: 'Add',
      data: {
        title : null,
        description : null
      }
    }
  },
  mounted() {
    this.getAll()
  },
  methods: {
    async getAll() {
      let response = await fetch('http://localhost:8000/api/posts');
      this.posts = await response.json();
    },
    async edit(postID) {
      const post = this.posts.find(({id}) => id === postID)
      this.action = 'Update'
      for (let prop in post) {
        this.data[prop] = post[prop];
      }
    },
    async updateOrCreate() {
      if(this.action === 'Add') {
        try {
          let response = await fetch('http://localhost:8000/api/posts', {
            method: 'POST',
            body: JSON.stringify(this.data),
            headers: {
              'Content-Type': 'application/json'
            }
          });

          let { status, post } = await response.json();

          if(status) {
            this.posts.push(post)
            this.data = {
              title : null,
              description : null
            }

            alert('Post added!')
          } else {
            throw new Error('Something went wrong!');
          }

        } catch(err) {
          alert(err);
        }
      } else {
        let id = this.data.id;
        try {
          let response = await fetch(`http://localhost:8000/api/posts/${id}`, {
            method: 'PUT',
            body: JSON.stringify(this.data),
            headers: {
              'Content-Type': 'application/json'
            }
          });

          let { status, post } = await response.json();

          if(status) {
            this.posts = this.posts.map(el => el.id !== post.id ? el : post)
            this.data = {
              title : null,
              description : null
            }
            this.action = 'Add'
            alert('Post updated!')
          } else {
            throw new Error('Something went wrong!');
          }

        } catch(err) {
          alert(err);
        }
      }

    },
    async remove(post) {
      try {
        let response = await fetch(`http://localhost:8000/api/posts/${post}`, {
          method: "DELETE"
        });

        let { status } = await response.json();

        if(status) {
          this.posts = this.posts.filter(({id}) => id !== post)
          alert('Post deleted!')
        } else {
          throw new Error('Something went wrong!');
        }

      } catch(err) {
        alert(err);
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

header {
  position: relative;
}


h4, p {
  margin: 0;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

li {
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 2px;
  border: 1px solid #000;
  text-align: left;
  padding: 10px 15px;
  position: relative;
  margin-bottom: 10px;
}

.btn-box {
  position: absolute;
  right: 25px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
}

header .btn {
  position: absolute;
  right: 25px;
  top: 50%;
  transform: translateY(-50%);
}

footer form {
  display: flex;
  flex-direction: column;
  max-width: 30%;
  margin: 15px auto;
  gap: 10px;
}


.container {
  padding: 0 15px;
  max-width: 1140px;
  width: 100%;
  margin: 0 auto;
}
</style>
