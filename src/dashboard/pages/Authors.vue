<template>
  <b-container class="page  animate__animated animate__fadeIn">
    <h2>Manage Authors</h2>
    <p>Authors: {{ authors.length }}, Books: {{ books.length }}</p>

    <b-row>
      <b-col lg="6" class="my-1">
        <b-button v-b-modal.modal-1 id="createbutton" variant="success"
          >Add Author</b-button
        >
      </b-col>

      <b-col lg="6" class="my-1">
        <b-form-group
          label="Filter"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filter"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filter" @click="filter = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>
    </b-row>

    <b-table
      striped
      hover
      :items="authors"
      :fields="fields"
      :filter="filter"
      @filtered="onFiltered"
      :current-page="currentPage"
      :per-page="perPage"
      @row-clicked="onRowClicked"
    >
      <template #cell(numbooks)="row">
        {{ books.filter(item => item.author_id == row.item.id).length }}
      </template>
      <template #cell(actions)="row">
        <b-button
          size="sm"
          @click="onClickDelete(row.item.id)"
          class="mr-1"
          variant="danger"
        >
          Delete
        </b-button>
      </template>
    </b-table>
    <b-row>
      <b-col lg="6" class="my-1"> </b-col>

      <b-col lg="6" class="my-1">
        <b-pagination
          v-model="currentPage"
          :total-rows="totalRows"
          :per-page="perPage"
          align="fill"
          size="sm"
          class="my-0"
        ></b-pagination>
      </b-col>
    </b-row>

    <b-modal id="modal-1" title="Add Author" hide-footer>
      <b-form-group id="input-group-1" label="Author Name:">
        <b-form-input
          placeholder="John Doe"
          required
          v-model="modal_name"
        ></b-form-input>
      </b-form-group>
      <b-button class="mt-3" @click="addAuthor()">Add</b-button>
    </b-modal>
    <!-- detail modal -->
    <b-modal id="modal-2" ref="modal-2" title="Author & Books" hide-footer>
      <b-form-group id="input-group-1" label="Author Name">
        <b-form-input
          placeholder="John Doe"
          required
          v-model="modal2_name"
        ></b-form-input>
      </b-form-group>
      <h5>Books</h5>
      <li v-for="book in modal2_books">
       {{book.title}}, {{book.pages}}P
      </li>
     
      <!-- <b-button class="mt-3">Edit</b-button> -->
    </b-modal>
  </b-container>
</template>
<script>
import HeartIcon from "vue-ionicons/dist/md-heart.vue";
import axios from "axios";

export default {
  components: { HeartIcon },
  data() {
    return {
      showLoader: true,
      showErrALert: false,
      tagVariants: ["primary", "success", "warning", "info", "dark", "danger"],
      authors: [],
      books: [],
      items: [
        { age: 40, first_name: "Dickerson", last_name: "Macdonald" },
        { age: 21, first_name: "Larsen", last_name: "Shaw" },
        { age: 89, first_name: "Geneva", last_name: "Wilson" },
        { age: 38, first_name: "Jami", last_name: "Carney" }
      ],
      fields: [
        {
          key: "name",
          label: "Author Name",
          sortable: true,
          sortDirection: "desc"
        },
        {
          key: "numbooks",
          label: "Number of Books",
          sortable: true,
          class: "text-center"
        },
        { key: "actions", label: "Actions" }
      ],
      filter: null,
      totalRows: 1,
      currentPage: 1,
      perPage: 5,
      // add modal
      modal_name: "",
      //edit modal
      modal2_name: "",
      modal2_books: []
    };
  },
  head: {
    title: "Authors"
  },
  methods: {
    getAuthors(success, error) {
      axios
        .get("http://localhost:8000/authors")
        .then(res => {
          console.log("##########authors", res);
          res.data.forEach(item => {
            this.authors.push({
              id: item.id,
              name: item.name
            });
          });
          this.totalRows = this.authors.length;
          success();
        })
        .catch(err => {
          error();
          console.log("Error fetching data " + err);
        });
    },
    getBooks(success, error) {
      axios
        .get("http://localhost:8000/books")
        .then(res => {
          console.log("##########books");
          res.data.forEach(item => {
            this.books.push(item);
          });
          success();
        })
        .catch(err => {
          error();
          console.log("Error fetching data " + err);
        });
    },
    addAuthor() {
      axios
        .post("http://localhost:8000/authors", {
          id: 0,
          name: this.modal_name
        })
        .then(response => {
          console.log(response.data);
          window.location.reload();
          // this.$bvModal.hide("his.$bvModal.hide()");
        })
        .catch(error => {
          console.log(error);
        });
    },
    onRowClicked(item) {
      this.modal2_name = item.name;
      this.modal2_books = this.books.filter(book => book.author_id == item.id);
      this.$refs["modal-2"].show();
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
    onClickDelete(author_id) {
      axios
        .post("http://localhost:8000/authors/del/" + author_id, {})
        .then(response => {
          console.log(response.data);
          window.location.reload();
        })
        .catch(error => {
          console.log(error);
        });
    }
  },
  async mounted() {
    this.getAuthors(
      () => {
        this.showLoader = false;
        this.getBooks(
          () => {
            this.showLoader = false;
          },
          () => {
            this.showErrALert = true;
            this.showLoader = false;
          }
        );
      },
      () => {
        this.showErrALert = true;
        this.showLoader = false;
      }
    );
  }
};
</script>
<style scoped>
.container {
  margin-top: 120px;
}
.row {
  margin-top: 40px;
}
.spinner-border {
  margin-top: 40px;
}
.card {
  text-align: left;
  color: #000;
}
#createbutton {
  margin-bottom: 10px;
}
</style>
