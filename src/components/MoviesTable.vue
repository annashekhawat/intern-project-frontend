<template>
  <div id="moviesTable">
    <template>
      <div>

        <div id="rightContainer3">
      <multiselect
               v-model="selectedValues2"
               :options="movies"
               :multiple="true"
               placeholder="Filter by movie"
               track-by="movie"
               label="movie"
               @select="filteredMovies">
             </multiselect>
           </div>
         <div id="rightContainer1">
           <multiselect
                    v-model="selectedValues1"
                    :options="directors"
                    track-by="director"
                    placeholder="Filter by director"
                    label="director"
                    @select="filteredDirectors"
                    :multiple="true">
            </multiselect>
          </div>

          <div id="rightContainer2">
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
      :items="filteredMovies"
      :fields="fields"
      :head-variant="headVariant"
      :table-variant="tableVariant"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc">

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
          Delete
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
        sortBy: 'movie_id',
        sortDesc: false,
        maxLengthOfReview: '120',
        fields: [
            {key: 'movie_id', label: 'Movie Id', sortable: true},
            {key: 'movie', label: 'Movie', sortable: true},
            {key: 'director', label: 'Director', sortable: true},
            {key: 'genre', label: 'Genre', sortable: true},
            {key: 'review', label: 'Review', sortable: true},
            {key: 'delete_this_movie', label: 'Delete Movie', sortable: false}],
        selectedValues: [],
        selectedValues1: [],
        selectedValues2: [],
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
        directors: [],
        movies: [],
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
        const directorList = [];
        if (this.selectedValues.length === 0)
        {
          this.directors = [];
          this.selectedValues1 = [];
          this.movies = [];
          this.selectedValues2 = [];
        }
        for (const item of this.items) {
          if(this.selectedGenres.includes(item.genre)) {
          directorList.push(item);
        }
      }
      this.directors = directorList;

      // return this.items.filter(item => this.selectedGenres.includes(item.genre));
      },
      selectedDirectors() {
        const director_list = [];
        for (const { director } of this.selectedValues1) {
          director_list.push(director)
        }
        return director_list
      },
      filteredDirectors() {
        const filteredDir = [];

        if(this.selectedValues.length === 0)
        {
          this.directors = [];
          // return this.items;
        }

        if (this.selectedValues1.length === 0)
        {
          this.movies = [];
        }
        for (const item of this.directors) {
          if(this.selectedDirectors.includes(item.director)) {
            filteredDir.push(item);
        }
      }
      this.movies = filteredDir;
    },
    selectedMovie() {
      const mov_list = [];
      for (const { movie } of this.selectedValues2) {
        mov_list.push(movie)
      }
      return mov_list
    },
    filteredMovies() {
      const filtered_movie_list = [];
      if(this.selectedValues.length === 0)
      {
        this.directors = [];
        this.movies = [];
        return this.items;
      }
      if(this.selectedValues1.length === 0)
      {
        this.movies = [];
        return this.directors;
      }

      if (this.selectedValues2.length === 0)
      {
        return this.movies;
      }
      for (const item of this.movies) {
        if(this.selectedMovie.includes(item.movie)) {
          filtered_movie_list.push(item);
      }
    }
    return filtered_movie_list;
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
#rightContainer1 {
   float:right;
}

#rightContainer2 {
   float:right;
}

#rightContainer3 {
   float:right;
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
