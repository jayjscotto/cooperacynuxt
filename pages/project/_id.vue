<template>
  <div class="project-page-container py-5 py-md-3 w-100">
    <div
      class="project-details-container d-flex flex-wrap align-items-center justify-content-around w-100 projectbox p-3"
    >
      <img
        :src="projectImage"
        class="project-details-image img-fluid img-responsive col-8 col-md-2 mb-2 mb-md-0"
      />
      <div
        class="project-details col-12 col-md-8 d-flex flex-column align-items-center justify-content-center text-center"
      >
        <h2 class="font-weight-light">{{ oneproject.name.toUpperCase() }}</h2>
        <small class="mb-2 up">{{ oneproject.brief }}</small>
        <small class="mb-2 up">
          <strong>STAGE: </strong> {{ stage(oneproject.stage) }}
        </small>
        <p class="justify">{{ oneproject.content }}</p>
        <p>
          <strong>PROJECT BASED IN:</strong>
          {{ location(oneproject.place) }}
        </p>
        <p>
          TAGS:
          <span v-for="tag in this.$store.state.tag" :key="tag.id">{{
            tag.name
          }}</span>
          <span
            >- <b-link v-b-modal.addtagmodal class="ac">Add</b-link> or
            <b-link v-b-modal.removetagmodal class="au">remove</b-link> a
            tag</span
          >
        </p>
      </div>
      <div class="project-stats d-flex flex-column col-8 col-md-2">
        <small>
          <strong>AFTF: </strong>{{ oneproject.anonymous ? 'ON' : 'OFF' }}
        </small>
        <small v-if="oneproject.stage != '5'">
          <strong>BUDGET: </strong>{{ Math.round(oneproject.collected) }}/{{
            Math.round(oneproject.budget)
          }}
        </small>
        <small v-if="oneproject.stage == '5'">
          <strong>FEE: </strong> {{ Math.round(oneproject.budget) }}
        </small>
        <small>
          <strong>HUDGET: </strong> {{ oneproject.participant }}/{{
            oneproject.hudget
          }}
        </small>
        <small>
          <strong>PARENT PROJECT: </strong> {{ oneproject.parent }}
        </small>
        <small>
          <strong>CATEGORY: </strong> {{ category(oneproject.category) }}
        </small>
      </div>
      <p>VOTE FOR THIS PROJECT:</p>
      <votebar
        :voteprop="oneproject"
        :proptype="'project'"
        :voteId="oneproject.id"
      />
      <div class="col-12 mt-4">
        <b-link class="au" v-if="oneproject.stage != 1" @click="edit"
          >Edit this project</b-link
        >
        <b-link
          class="ae"
          v-if="oneproject.stage != 1 && this.$auth.user.role == 1"
          @click="archive"
          >Archive this project</b-link
        >
        <b-link
          class="au"
          v-if="oneproject.stage == 1 && this.$auth.user.role == 1"
          @click="unarchive"
          >Resume this project</b-link
        >
        <b-link class="at" @click="copy">Copy this project</b-link>
      </div>
    </div>
    <div class="comments-container w-100 mt-5">
      <h3 class="font-weight-normal text-center">COMMENTS AND QUESTIONS</h3>
      <comment />
    </div>

    <!-- ADD TAGS MODAL -->
    <b-modal id="addtagmodal" title="Add a new tag" hide-header-close>
      <p class="my-4">
        (CURRENT TAGS:
        <span v-for="tag in this.$store.state.tag" :key="tag.id">{{
          tag.name
        }}</span
        >)
      </p>
      <b-form @submit.prevent="addtag">
        <b-form-group
          label-for="newtagInput"
          description="Insert a new tag for your project"
        >
          <span>{{ oneproject.name }}</span>
          <b-form-input id="newTagInput" v-model="formTag"></b-form-input>
        </b-form-group>
      </b-form>
      <template slot="modal-footer" slot-scope="{ ok, cancel }">
        <b-button size="sm" class="bcare" @click="addtag()">ADD</b-button>
        <b-button size="sm" class="btransparency" @click="cancel()"
          >CLOSE</b-button
        >
      </template>
    </b-modal>
    <!-- REMOVE TAGS MODAL -->
    <b-modal id="removetagmodal" title="Remove a tag" hide-header-close>
      <p class="my-4">Select a tag:</p>
      <b-form @submit.prevent="removetag">
        <b-form-group
          label-for="newtagInput"
          label="Tag:"
          description="Remove tags from your project"
        >
          <span>{{ oneproject.name }}</span>
          <b-form-select id="removeTagInput" v-model="formRemoveTag">
            <option
              v-for="tag in this.$store.state.tag"
              :key="tag.id"
              :value="tag.name"
              >{{ tag.name }}</option
            >
          </b-form-select>
        </b-form-group>
      </b-form>
      <template slot="modal-footer" slot-scope="{ ok, cancel }">
        <b-button size="sm" class="bcare" @click="removetag()">REMOVE</b-button>
        <b-button size="sm" class="btransparency" @click="cancel()"
          >CLOSE</b-button
        >
      </template>
    </b-modal>
  </div>
</template>

<script>
import votebar from '@/components/votebar'
import comment from '@/components/comment'

export default {
  middleware: ['auth'],
  validate({ params }) {
    // Must be a number
    return /^\d+$/.test(params.id)
  },
  head() {
    return {
      title: 'Cooperacy - Project: ' + this.oneproject.name
    }
  },
  components: { votebar: votebar, comment: comment },
  async fetch({ store, params }) {
    await store.dispatch('getCommentAction', {
      projectid: params.id,
      userid: store.state.auth.user.id,
      limit: ' LIMIT 1'
    })
    await store.dispatch('getTagAction', { projectid: params.id })
    await store.dispatch('getProjectAction', {
      projectid: params.id,
      userid: store.state.auth.user.id
    })
    await store.dispatch('getPlaceAction')
    await store.dispatch('getCountryAction')
  },
  data() {
    return {
      formTag: '',
      formRemoveTag: ''
    }
  },
  computed: {
    oneproject() {
      while (
        this.$store.state.project.findIndex(
          project => project.id == this.$route.params.id
        ) == -1
      ) {
        this.$store.dispatch('getProjectAction', {
          projectid: this.$route.params.id,
          userid: this.$store.state.auth.user.id
        })
      }
      return this.$store.state.project[
        this.$store.state.project.findIndex(
          project => project.id == this.$route.params.id
        )
      ]
    },
    projectImage() {
      try {
        return require('../../assets/image/project/' +
          this.oneproject.id +
          '.png')
      } catch (e) {
        return require('../../assets/image/project/0.png')
      }
    }
  },
  methods: {
    archive() {
      this.$store.dispatch('projectFormAction', {
        stage: 1,
        id: this.oneproject.id,
        name: this.oneproject.name,
        brief: this.oneproject.brief,
        content: this.oneproject.content,
        video: this.oneproject.video,
        anonymous: this.oneproject.anonymous,
        parent: this.oneproject.parent,
        budget: this.oneproject.budget,
        hudget: this.oneproject.hudget,
        country: this.oneproject.country,
        place: this.oneproject.place
      })
      this.$store.dispatch('getUserProjectAction', {
        userid: this.$auth.user.id
      })
      return this.$router.push({ path: '/user' })
    },
    edit() {
      this.$store.dispatch('editSwitchAction', { id: this.$route.params.id })
      return this.$router.push({ path: '/project/form' })
    },
    unarchive() {
      this.$store.dispatch('editSwitchAction', { id: this.$route.params.id })
      return this.$router.push({ path: '/project/form' })
    },
    copy() {
      this.$store.dispatch('editSwitchAction', {
        id: this.$route.params.id,
        copy: true
      })
      this.$store.dispatch('getUserProjectAction', {
        userid: this.$auth.user.id
      })
      return this.$router.push({ path: '/project/form' })
    },
    async addtag() {
      if (this.formTag) {
        this.formTag = this.formTag.replace(/^[ ]+/gi, '') //removes spaces at the beginning of tag
        await this.$store
          .dispatch('tagFormAction', {
            project: this.oneproject.id,
            name: this.formTag,
            tag: 'add'
          })
          .catch(err => {
            console.error(err)
          })
        this.formTag = ''
        this.$toast.success('Added!', { duration: 1000, className: 'toasts' })
      }
    },
    async removetag() {
      if (this.formRemoveTag) {
        alert(this.formRemoveTag)
        await this.$store
          .dispatch('removeTagAction', {
            project: this.oneproject.id,
            name: this.formRemoveTag
          })
          .catch(err => {
            console.error(err)
          })
      }
      this.formRemoveTag = ''
      this.$toast.success('Removed!', { duration: 1000, className: 'toasts' })
    },
    stage(id) {
      return this.$store.state.stage.find(stage => stage.id == id).name
    },
    category(id) {
      return this.$store.state.category.find(category => category.id == id).name
    },
    location(id) {
      let projectPlace = this.$store.state.place.find(place => place.id == id)
      if (projectPlace) {
        let projectCountry = this.$store.state.country.find(
          country => country.id == projectPlace.country
        )
        if (projectCountry) {
          return projectPlace.name + ', ' + projectCountry.name
        }
      } else {
        return 'Unknown'
      }
    }
  }
}
</script>
