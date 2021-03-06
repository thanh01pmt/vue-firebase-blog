<template>
  <div id="add-post" class="section">
    
    <div v-show="!loggedIn" class="container">
      <div class="columns is-centered">
        <form class="column is-one-third" v-on:submit.prevent="trySignIn">
          <h1 class="title">Log In</h1>
          <div class="field">
            <label for="email">Email</label>
            <div class="control">
              <input class="input" type="email" name="email" v-model="userEmail">
            </div>
          </div>
          <div class="field">
            <label for="password">Password</label>
            <div class="control">
              <input class="input" type="password" name="password" v-model="password">
            </div>
          </div>
          <div class="field">
            <div class="control">
              <button class="button">Log In</button>
              <p class="help is-danger" v-show="hasError">{{errorMessage}}</p>
            </div>
          </div>
        </form>
      </div>
    </div>
    
    <div v-show="loggedIn">
      
      <div class="container">
        <div class="columns is-centered">
          <div class="column is-half">
            <h1 class="title">Create New Post!</h1>
            <form v-on:submit.prevent="createPost">
              <div class="field">
                <label for="title">Title</label>
                <div class="control">
                  <input class="input" type="text" name="title" v-model="title">
                </div>
              </div>
              <div class="field">
                <label for="category">Category</label>
                <div class="control">
                  <div class="select">
                    <select name="category" v-model="category">
                      <option v-for="category in categories">{{category}}</option>
                    </select>
                  </div>
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <label class="radio">
                    <input type="radio" name="postorlink" value="post" v-model="postOrLink">
                    Post
                  </label>
                  <label class="radio">
                    <input type="radio" name="postorlink" value="link" v-model="postOrLink">
                    Link
                  </label>
                </div>
              </div>
              <div class="field" v-show="postOrLink == 'post'">
                <label class="label">Content</label>
                <div class="control">
                  <textarea class="textarea" placeholder="### Use Markdown" v-model="content"></textarea>
                </div>
              </div>
              <div class="field" v-show="postOrLink == 'link'">
                <label for="url">Url</label>
                <div class="control">
                  <input class="input" type="text" name="url" v-model="content">
                </div>
              </div>
              <div class="field">
                <label for="thumbnail">Thumbnail</label>
                <div class="control">
                  <input class="input" type="text" name="thumbnail" v-model="thumbnail">
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <button class="button">Add Post</button>
                  <p class="help is-danger" v-show="hasError">{{errorMessage}}</p>
                </div>
              </div>
            </form>
            <div class="notification is-primary" v-show="success">Post Added!</div>
          </div>
        </div>
      </div>

      <div class="container">
        <div class="columns is-centered">
          <div class="column is-half">
            <form v-on:submit.prevent="uploadFile">
              <h3 class="subtitle is-3">Add Images</h3>
              <div class="field">
                <div class="file has-name">
                  <label class="file-label">
                    <input class="file-input" type="file" name="resume" v-on:change="onFileChange">
                    <span class="file-cta">
                      <span class="file-icon">
                        <i class="fas fa-upload"></i>
                      </span>
                      <span class="file-label">
                        Choose an image…
                      </span>
                    </span>
                    <span class="file-name">{{fileName}}</span>
                  </label>
                </div>
              </div>
              <div class="field">
                <div class="control">
                  <button class="button">Upload Image</button>
                  <p class="help is-danger" v-show="hasError">{{errorMessage}}</p>
                </div>
              </div>
            </form>
            <ul style="margin-top: 2em;">
              <li v-for="fileUrl in fileUrls">
                <div class="card">
                  <div class="card-image">
                    <figure class="image">
                      <img v-bind:src="fileUrl" alt="image" >
                    </figure>
                  </div>
                  <div class="card-content"><p><em>{{fileUrl}}</em></p></div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="container">
        <div class="columns is-centered">
          <div class="column is-half">
            <p><a href="#" v-on:click.prevent="trySignOut">Log out</a></p>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>

import firebase from 'firebase'

export default {
  name: 'addpost',
  data () {
    return {
      loggedIn: false,
      hasError: false,
      success: false,
      errorMessage: '',
      userEmail: '',
      password: '',
      categories: ['personal', 'work', 'thoughts', 'fun'],
      title: '',
      content: '',
      thumbnail: '',
      category: 'personal',
      postOrLink: 'post',
      fileName: '',
      inputFile: {},
      fileUrls: []
    }
  },
  created () {
    this.createAuthListener()
  },
  methods: {
    handleError (msg) {
      this.hasError = true
      this.errorMessage = msg
    },
    reset () {
      this.loggedIn = false
      this.error =  false
      this.errorMessage = ''
      this.userEmail = ''
      this.password = ''
    },
    userSignedIn (user) {
      this.loggedIn = true
      this.error = false
      this.errorMessage = ''
    },
    createAuthListener () {
      let _this = this

      firebase.auth().onAuthStateChanged(function (user) {
        if (user) {
          _this.userSignedIn(user)
        } else {
          // No user is signed in.
        }
      })
    },
    trySignIn () {
      firebase
        .auth()
        .signInWithEmailAndPassword(this.userEmail, this.password)
        .catch((error) => {
          this.handleError(error.message)
        })
    },
    trySignOut () {
      firebase
        .auth()
        .signOut()
        .then(() => {
          // Sign-out successful.
          this.reset()
        }).catch((error) => {
          // An error happened.
          this.handleError(error.message)
        })
    },
    createPost () {
      let newPostRef = firebase
        .database()
        .ref('posts')
        .push({
          title: this.title,
          category: this.category,
          type: this.postOrLink,
          content: this.content,
          thumbnail: this.thumbnail,
          postDate: Date.now()
        })

      firebase
        .database()
        .ref('categories/' + this.category + '/' + newPostRef.key)
        .set(true)

      this.success = true
    },
    onFileChange (e) {
      const input = event.target
      
      if (input.files && input.files[0]) {
        this.inputFile = input.files[0]
        this.fileName = this.inputFile.name
      }
    },
    addImageToList (url) {
      this.fileUrls.push(url)
    },
    uploadFile () {
      if (this.inputFile.name.length === 0) {
        return
      }

      const storageRef = firebase.storage().ref()
      const uploadTask = storageRef.child('images/' + this.inputFile.name).put(this.inputFile)

      uploadTask.on(firebase.storage.TaskEvent.STATE_CHANGED,
        (snapshot) => {
          const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100
          
          console.log('Upload is ' + progress + '% done');
          
          switch (snapshot.state) {
            case firebase.storage.TaskState.PAUSED:
              console.log('Upload is paused');
              break;
            case firebase.storage.TaskState.RUNNING:
              console.log('Upload is running');
              break;
          }

        }, (error) => {
          this.handleError(error.message)
        }, () => {
          // Upload completed successfully, now we can get the download URL
          this.addImageToList(uploadTask.snapshot.downloadURL)
        })
    }
  }
}
</script>

<style lang="sass" scoped>
  @import "../../node_modules/bulma/bulma.sass";
</style>
