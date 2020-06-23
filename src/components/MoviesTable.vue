<template>
  <div>
    <b-table id="moviesTable"
      :striped="striped"
      :bordered="bordered"
      :borderless="borderless"
      :outlined="outlined"
      :small="small"
      :hover="hover"
      :dark="dark"
      :fixed="fixed"
      :foot-clone="footClone"
      :no-border-collapse="noCollapse"
      :items="items"
      :fields="fields"
      :head-variant="headVariant"
      :table-variant="tableVariant">
      <template v-slot:cell(review)="row">
        
        
        <b-button id = "add-review-button" v-if="row.item.isEmpty && !row.item.editingReview" size="sm" variant="primary"  v-on:click="row.item.editingReview = true;" class="mr-2">
          Add Review
        </b-button>
        <b-form-input v-if="!row.item.isEmpty || row.item.editingReview" autofocus = "true" v-sync="row.item.review" v-model="row.item.review" placeholder="Add review here" ></b-form-input>
        <b-button id = "submit-review-button" v-if="row.item.editingReview" size="sm" variant="success"  v-on:click="row.item.editingReview = false; row.item.review == '' ? row.item.isEmpty = true : row.item.isEmpty = false;" class="mr-2">
          Submit
        </b-button>
      </template>
      </b-table>
      
  </div>
</template>

<script>

var datalist = [
          { isEmpty: false, editingReview: false, movie_id: '1', movie: 'Endgame', director: 'Anthony Russo', genre: 'Action', review: 'Very good' },
          { isEmpty: true, editingReview: false, movie_id: '2', movie: 'Avengers: Age of Ultron', director: 'Joss Whedon', genre: 'Action', review: '' },
          { isEmpty: false, editingReview: false, movie_id: '3', movie: 'The Avengers', director: 'Joe Russo', genre: 'Superhero', review: 'Good' },
          { isEmpty: false, editingReview: false, movie_id: '4', movie: 'Get Out', director: 'Jordan Peele', genre: 'Horror', review: '' }
        ];
  export default {
    data() {
      return {
        fields: [
            {key: 'movie_id', label: 'Movie Id', sortable: true},
            {key: 'movie', sortable: true},
            {key: 'director', sortable: true},
            {key: 'genre', sortable: true},
            {key: 'review', sortable: true}],
        items: datalist,
        tableVariants: [
          'primary',
          'secondary',
          'info',
          'danger',
          'warning',
          'success',
          'light',
          'dark'
        ],
        striped: true,
        bordered: true,
        borderless: false,
        outlined: false,
        small: false,
        hover: false,
        dark: false,
        fixed: false,
        footClone: false,
        headVariant: null,
        tableVariant: '',
        noCollapse: false
      }
    },
    methods: {
      greet: function (event) {

      // `this` inside methods points to the Vue instance
      alert('Hello ' + this.name + '!')
      // `event` is the native DOM event
      if (event) {
        alert(event.target.tagName)
      }
    }
    }
  }
</script>

<style lang="scss" scoped>
#moviesTable {
    width: 90%;
    margin: auto;
    margin-top: 5em;
}

#submit-review-button {
  margin-top: 0.5em;
}
</style>

