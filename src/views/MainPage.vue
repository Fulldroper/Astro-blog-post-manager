<script>
export default {
  data() {
    return {
      branch: window.localStorage.getItem("Branch") || false,
      username: window.localStorage.getItem("Github username") || false,
      path: window.localStorage.getItem("Path") || false,
      repo: window.localStorage.getItem("Repo name") || false,
      token: window.localStorage.getItem("Token") || false,
      regImg: /!\[.*\]\((\..*)\)/gm ,
      posts: [],
      filtered: [],
      url: "",
      selected: "",
      preview: false
    };
  },
  mounted: async function() {
    this.loadList()
    
    const externalScript = document.createElement('script')
    externalScript.setAttribute('src', 'https://cdn.jsdelivr.net/npm/marked/marked.min.js')
    document.head.appendChild(externalScript)

    this.titleSize()
  },
  methods: {
    generateUrl() {
      if (
        !this.branch ||
        !this.username ||
        !this.path ||
        !this.repo ||
        !this.token
      ) return false;

      return `https://api.github.com/repos/${this.username}/${this.repo}/contents/${this.path}?ref=${this.branch}`
    },
    filter({target}) {
      this.filtered = this.posts.filter(post => post.name.startsWith(target.value))
    },
    previewSwich() {
      this.preview = !this.preview
    },
    async edit(file) {
      const post = this.posts.find(post => post.name == file)
      const file_url = post.type == 'file' ? post.download_url : `https://raw.githubusercontent.com/${this.username}/${this.repo}/main/${this.path}/${file}/index.md`

      try {
        const response = await fetch(file_url);
        if (response.ok) {
          this.$refs.editor.value = await response.text();
          this.selected = post
          this.renderFile()
          this.titleSize()
        }
      } catch (error) {
        console.log('File does not exist, creating new one', error);
      }
    },
    async loadList() {
      if (!(this.url = this.generateUrl())) return;
      try {
        const response = await fetch(this.url, {
          headers: {
            'Authorization': `token ${this.token}`
          }
        });
        if (response.ok) {
          this.posts = await response.json();
          this.filtered = this.posts;
        }
      } catch (error) {
        console.log('File does not exist, creating new one', error);
      }
    },
    titleSize() {
      this.$refs['file-title'].size = this.$refs['file-title'].value.length
    },
    getTextWidth(text, font) {
    // Create a canvas element
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");

    // Set the font. The format is similar to CSS font property, e.g., "16px Arial"
    context.font = font;

    // Measure the text width
    const metrics = context.measureText(text);
    return metrics.width;
    },
    renderFile() {
      this.$refs.preview.innerHTML = window.marked.parse(
        this.imagePathModify(
          this.regImg,
          this.$refs.editor.value,
          `https://raw.githubusercontent.com/${this.username}/${this.repo}/${this.branch}/${this?.selected?.path}`
        )
      )
    },
    clearSelected () {
      this.$refs.editor.value = "";
      this.selected = ""
      this.$refs.preview.innerHTML=""
      this.titleSize()
    },
    async putFile () {
      try {
        // Fetch the current file's SHA
        const shaResponse = await fetch(`https://api.github.com/repos/${this.username}/${this.repo}/contents/${this.$refs['file-title'].value}`, {
          headers: {
            'Authorization': `token ${this.token}`,
            'Content-Type': 'application/json',
            'X-GitHub-Api-Version': '2022-11-28'
          }
        });

        let sha = null;
        if (shaResponse.ok) {
          const shaData = await shaResponse.json();
          this.selected.sha = shaData.sha;
        }
        const response = await fetch(`https://api.github.com/repos/${this.username}/${this.repo}/contents/${this.$refs['file-title'].value}`, {
          method: 'PUT',
          headers: {
            'Authorization': `token ${this.token}`,
            'Content-Type': 'application/json',
            'X-GitHub-Api-Version': '2022-11-28'
          },
          body: JSON.stringify({
            message: `Adding/Updating file ${this.$refs['file-title'].value}`,
            content: btoa(this.$refs['editor'].value),
            sha: this.selected.sha || null,
            branch: this.branch,
            owner: this.username,
            repo: this.repo,
            path: this.$refs['file-title'].value,
            committer: {
              name: this.username,
              email: this.username +'@github.com'
            },
          })
        });
        if (response.ok) {
          console.log('saved successfully');
        }
      } catch (error) {
        console.log('File does not exist, creating new one', error);
      }
    },
    async delFile() {
      try {
        const response = await fetch(`https://api.github.com/repos/${this.username}/${this.repo}/contents/${this.$refs['file-title'].value}`, {
          method: 'DELETE',
          headers: {
            'Authorization': `token ${this.token}`,
            'Content-Type': 'application/json',
            'X-GitHub-Api-Version': '2022-11-28'
          },
          body: JSON.stringify({
            message: `Deleting file ${this.$refs['file-title'].value}`,
            content: btoa(this.$refs['editor'].value),
            sha: this.selected.sha || null,
            branch: this.branch,
            owner: this.username,
            repo: this.repo,
            path: this.$refs['file-title'].value,
            committer: {
              name: this.username,
              email: this.username +'@github.com'
            },
          })
        });
        if (response.ok) {
          console.log('delete successfully');
        }
      } catch (error) {
        console.log('File does not exist, creating new one', error);
      }
    },
    addImage() {
      this.$refs.imageUpload.click();
    },
    toBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = reject;
      })
    },
    async fileUpload({target}) {
      for (const file of target.files) {
        try {
          const path = this.$refs['file-title'].value.split("/").slice(0,-1).join("/") + "/" + file.name
          const content = await this.toBase64(file);
          const response = await fetch(`https://api.github.com/repos/${this.username}/${this.repo}/contents/${path}`, {
            method: 'PUT',
            headers: {
              'Authorization': `token ${this.token}`,
              'Content-Type': 'application/json',
              'X-GitHub-Api-Version': '2022-11-28',
            },
            body: JSON.stringify({
              message: `Adding/Updating file ${path}`,
              content: content.split(',')[1],
              sha: this.selected.sha || null,
              branch: this.branch,
              owner: this.username,
              repo: this.repo,
              path,
              committer: {
                name: this.username,
                email: this.username +'@github.com'
              },
            })
          });
          if (response.ok) {
            console.log('saved successfully');
          }
        } catch (error) {
          console.log('File does not exist, creating new one', error);
        }
      }
    },
    imagePathModify(r, v, n) {
      let text = v
      const matched = [...v.matchAll(r)]
      matched.forEach(m => {
        text = text.replace(m[0], 
          m[0].replace(m[1], n + m[1].slice(1))
        )
      }) 
      return text
    }
  }
}
</script>

<template>
  <main class="flex flex-row w-screen h-screen" v-if="this.url">
    <div class="flex flex-col py-2 bg-zinc-900 w-52">
      <div class="flex flex-col justify-center w-full p-2">
        <RouterLink to="/settings" class="cursor-pointer text-center text-base bg-zinc-600 hover:bg-zinc-500 text-white rounded p-2 block w-full">Settings</RouterLink>
        <input @input="filter" class="mt-2 bg-zinc-600 text-white rounded px-2 py-1 text-md" placeholder="Filter" type="text">
      </div>
      <div v-if="this.url" @click="edit(post.name)" class=" text-blue-100 px-3 cursor-pointer text-sm hover:bg-red-400" v-for="post of filtered">
        {{ post.name.replace(".md", "") }}
      </div>
      <div v-else>Please setup env in  settings</div>
    </div>
    <div class="w-full h-full bg-zinc-700 flex flex-col">
      <div class="text-zinc-200 flex flex-row justify-between w-full">
        <div class="w-max px-4 py-1 flex flex-row">
          <div @click="putFile" class="cursor-pointer w-max py-1 px-4 hover:text-zinc-400">Save</div>
          <div @click="addImage" class="cursor-pointer w-max py-1 px-4 hover:text-zinc-400">Add Image</div>
          <input
            ref="imageUpload"
            type="file"
            @change="fileUpload"
            accept="image/*"
            multiple="true"
            capture
            class="hidden"
          />
          <div class="cursor-pointer w-max py-1 px-4 hover:text-zinc-400" @click="previewSwich">Preview</div>
        </div>
        <input ref="file-title" @input="titleSize" class="bg-transparent w-full text-center" :value="selected ? (selected.type == 'file' ? selected.path : selected.path + '/index.md') : this.path + '/new_file.md'" ></input>
        <div class="w-max px-4 py-1 flex flex-row">
          <div v-if="selected.path" class="cursor-pointer w-max py-1 px-4 hover:text-zinc-400" @click="delFile">Delete</div>
          <div v-if="selected.path" class="cursor-pointer w-max py-1 px-4 hover:text-zinc-400" @click="clearSelected">Close</div>
        </div>
      </div>
      <div class="flex flex-row w-full h-full">
        <textarea @input="renderFile" ref="editor" style="resize: none;max-height: calc(100% - 40px);" class="bg-zinc-800 w-full h-full p-2 text-blue-100"></textarea>
        <div style="max-height: calc(100% - 40px);" :class="`w-full h-full bg-zinc-800 text-zinc-200 overflow-y-auto p-2 overflow-x-hidden border-l border-zinc-600 ${this.preview ? '' : 'hidden'}`" ref="preview"></div>
      </div>
    </div>
  </main>
  <main v-else class="flex flex-row w-screen h-screen items-center justify-center bg-zinc-900">
    <div class="flex flex-col justify-center w-max h-max p-2">
      <div class="cursor-pointer w-max py-1 px-4 text-zinc-200 text-3xl mb-4">Please setup env</div>
      <RouterLink to="/settings" class="cursor-pointer text-center text-xl bg-zinc-600 hover:bg-zinc-500 text-white rounded p-2 block w-full">Settings</RouterLink>
    </div>
  </main>
</template>

<style scoped></style>