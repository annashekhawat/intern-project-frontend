<template>
  <div id="moviesTable">
    <template>
    <div>

    <multiselect
             v-model="selectedValues"
             :options="options"
             :multiple="true"
             placeholder="Filter by genre"
             track-by="id"
             label="genre"
             @select="filteredGenres">
           </multiselect>
      </div>
    </template>
      <b-table
      fixed
      responsive
      :striped="striped"
      :bordered="bordered"
      :borderless="borderless"
      :outlined="outlined"
      :small="small"
      :hover="hover"
      :dark="dark"
      :foot-clone="footClone"
      :no-border-collapse="noCollapse"
      :items="filteredGenres"
      :fields="fields"
      :head-variant="headVariant"
      :table-variant="tableVariant">

      <!-- Code to set column widths individually-->
      <template v-slot:table-colgroup="scope">
        <col
          v-for="field in scope.fields"
          :key="field.key"
          :style="{ width: field.key === 'review' ? '30%' : (field.key === 'movie_id' ? '10%' : '15%') }"
        >
      </template>

      <!-- End Code to set column widths individually-->

      <!-- search box for search by genre -->
      <!-- <template slot="top-row" slot-scope="{ fields }">
        <td v-for="field in fields" :key="field.key">
          <input v-if= "field.key === 'genre'" v-model="filters[field.key]" :placeholder="field.label">
        </td>
      </template> -->

      <!--end code for the search feature -->

      <!-- Special code for the column 'Review'-->
      <template v-slot:cell(review)="row">

        <!-- Add review button-->
        <span class="d-inline-block" v-if="showAddReviewButton(row)" tabindex="0" v-b-tooltip.bottom v-bind:title="row.item.authorizedToEditReview ? 'Add a review for this movie' : 'You are not authorized to add this review'">
        <b-button id = "add-review-button" v-if="showAddReviewButton(row)" size="sm" variant="primary" v-bind:disabled="!row.item.authorizedToEditReview"  v-on:click="addReviewClicked(row)" class="mr-2">
          Add Review
        </b-button>
        </span>
        <!-- End Add review button-->

        <!-- Input field for review-->
        <b-form-input v-if="showSubmitReviewButton(row)" autofocus="true" v-model="row.item.review" :maxlength="maxLengthOfReview" placeholder="Add review here" ></b-form-input>
        <!-- End Input field for review-->

        <!-- Showing remaining characters for review-->
        <div v-if="showSubmitReviewButton(row)" class="input-group-addon" v-text="(maxLengthOfReview - row.item.review.length) + ' of ' + maxLengthOfReview + ' characters left'"></div>
        <!-- End Showing remaining characters for review-->

        <!-- Submit review button-->
        <b-button id = "submit-review-button" v-if="showSubmitReviewButton(row)" size="sm" variant="success"  v-on:click="submitReviewClicked(row)" v-b-tooltip.hover.bottom="'Submit this review'" class="mr-2">
          Submit
        </b-button>
        <!-- End Submit review button-->

        <!-- Label where review is displayed while it is not being edited-->
        <b-form-group id="review-label" v-if="showReviewLabel(row)" v-sync="row.item.review" v-bind:label="row.item.review"/>
        <!-- End Label where review is displayed while it is not being edited-->

        <!-- Edit review button-->
        <span class="d-inline-block" v-if="showEditReviewButton(row)" tabindex="0" v-b-tooltip.bottom v-bind:title="row.item.authorizedToEditReview ? 'Edit this review' : 'You are not authorized to edit this review'">
        <b-button id = "edit-review-button" v-if="showEditReviewButton(row)" size="sm" variant="outline-primary" v-bind:disabled="!row.item.authorizedToEditReview" v-on:click="editReviewClicked(row)" class="mr-2">
          <b-icon icon="pencil" aria-hidden="false"></b-icon>
        </b-button>
        </span>
        <!-- End Edit review button-->

      </template>
      <!-- End Special code for the column 'Review'-->
        

      <!-- Special code for the column 'Delete this Movie'-->
      <template v-slot:cell(delete_this_movie)="row">
        <!-- Delete review button-->
        <span class="d-inline-block" tabindex="0" v-b-tooltip.bottom v-bind:title="row.item.authorizedToEditReview ? 'Delete this movie' : 'You are not authorized to delete this movie'">
        <b-button id = "delete-review-button" sm="1" size="sm" variant="outline-danger"  v-bind:disabled="!row.item.authorizedToEditReview" v-on:click="deleteReviewClicked(row)" class="mr-2">
          <!--b-icon icon="trash-fill" aria-hidden="false"></b-icon-->
          Delete this Movie
        </b-button>
        </span>
        <!-- End Delete review button-->
      </template>
      <!-- End Special code for the column 'Delete this Movie'-->

    </b-table>
  </div>
</template>
<script src="https://unpkg.com/vue-multiselect@2.1.0"></script>
<script>
  import Multiselect from 'vue-multiselect'

  var dataList = []
  var authorizedGenres = [];

  function getGenresForRoles() {
    var xmlHttp = new XMLHttpRequest();
    xmlHttp.open( "GET", "http://localhost:8001/role/critic", false ); // false for synchronous request
    xmlHttp.send( null );
    var data = JSON.parse(xmlHttp.responseText);
    authorizedGenres = data;
  }

  function getAllRecords() {
    var xmlHttp = new XMLHttpRequest();
    xmlHttp.open( "GET", "http://localhost:8001/", false ); // false for synchronous request
    xmlHttp.send( null );
    var data = JSON.parse(xmlHttp.responseText);
    for(var i = 0; i < data.length; i++) {
      var dat = data[i];
      var dataPoint = {};
      dataPoint.editingReview = false;
      if(authorizedGenres.includes(dat.Genre)) {
        dataPoint.authorizedToEditReview = true;
      }
      else {
        dataPoint.authorizedToEditReview = false;
      }
      dataPoint.movie_id = dat.MovieId;
      dataPoint.movie = dat.Movie;
      dataPoint.director = dat.Director;
      dataPoint.genre = dat.Genre;
      dataPoint.review = dat.Review;
      dataList.push(dataPoint);
    }
  }
  getGenresForRoles();
  getAllRecords();

  // var dataList = [
  //           { editingReview: false, authorizedToEditReview: true, movie_id: '1', movie: 'Endgame', director: 'Anthony Russo', genre: 'Action', review: '' },
  //           { editingReview: false, authorizedToEditReview: false, movie_id: '2', movie: 'Avengers: Age of Ultron', director: 'Joss Whedon', genre: 'Action', review: '' },
  //           { editingReview: false, authorizedToEditReview: true, movie_id: '3', movie: 'The Avengers', director: 'Joe Russo', genre: 'Superhero', review: 'Quick-witted and nuanced, this movie takes the best of the genre -- iconic heroes fighting for truth and justice.' },
  //           { editingReview: false, authorizedToEditReview: true, movie_id: '4', movie: 'Get Out', director: 'Jordan Peele', genre: 'Horror', review: 'Bad' }
  //         ];
  export default {
    components: { Multiselect },
    data() {
      return {
        maxLengthOfReview: '120',
        fields: [
            {key: 'movie_id', label: 'Movie Id', sortable: true},
            {key: 'movie', label: 'Movie', sortable: true},
            {key: 'director', label: 'Director', sortable: true},
            {key: 'genre', label: 'Genre', sortable: true},
            {key: 'review', label: 'Review', sortable: true},
            {key: 'delete_this_movie', label: 'Delete this Movie', sortable: false}],
        selectedValues: [],
        options: [
          {'genre' : 'comedy','id' : '1'},
          {'genre' : 'horror', 'id' : '2'},
          {'genre' : 'sci-fi', 'id': '3'},
          {'genre' : 'drama', 'id': '4'},
          {'genre' : 'docu', 'id': '5'},
          {'genre' : 'period', 'id': '6'},
          {'genre' : 'rom-com', 'id': '7'},
        ],
        items: dataList,
        tableVariants: [
          "primary",
          "secondary",
          "info",
          "danger",
          "warning",
          "success",
          "light",
          "dark"
        ],
        striped: true,
        bordered: true,
        borderless: false,
        outlined: false,
        small: false,
        hover: false,
        dark: false,
        footClone: false,
        headVariant: null,
        tableVariant: '',
        noCollapse: false
      }
    },
    methods: {
      // Add review functionality
      showAddReviewButton: function(row) {
        return row.item.review == "" && !row.item.editingReview;
      },
      addReviewClicked: function(row) {
        row.item.editingReview = true;
      },

      // Edit review functionality
      showEditReviewButton: function(row) {
        return row.item.review != "" && !row.item.editingReview;
      },
      editReviewClicked: function(row) {
        row.item.editingReview = true;
      },

      // Submit review functionality
      showSubmitReviewButton: function(row) {
        return row.item.editingReview;
      },
      submitReviewClicked: function(row) {
        row.item.editingReview = false;
        this.$http.put("http://localhost:8001/genre/" + row.item.genre + "/directors/" + row.item.director + "/movies/" + row.item.movie,{
          movieID: row.item.movie_id,
          director: row.item.director,
          movie: row.item.movie,
          genre: row.item.genre,
          review: row.item.review
        }).then(function(data){
          return data.json();
        });
      },

      // Delete review functionality
      deleteReviewClicked: function(row) {
        var reallyDelete = confirm("Are you sure you want to delete this movie?");
        if(!reallyDelete) return;
        var xmlHttp = new XMLHttpRequest();
        xmlHttp.open( "DELETE",
          "http://localhost:8001/genre/" + row.item.genre + "/directors/" + row.item.director + "/movies/" + row.item.movie,
          false ); // false for synchronous request
        xmlHttp.send( null );
        var data = JSON.parse(xmlHttp.responseText);
        console.log(data);
        location.reload();
      },

      //Review label functionality
      showReviewLabel: function(row) {
        return row.item.review != "" && !row.item.editingReview;
      }
    },
    computed: {
      selectedGenres() {
        const genres = [];
        for (const { genre } of this.selectedValues) {
          genres.push(genre)
        }
        return genres
      },
      filteredGenres() {
        if (this.selectedValues.length === 0) return this.items

        return this.items.filter(item => this.selectedGenres.includes(item.genre));
      }
    }
  };
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

#review-label {
  margin-bottom: 0em;
}

.input-group-addon {
  margin-top: 0.2rem;
  font-size: 0.85rem;
  color: #6f6f6f;
}

</style>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
