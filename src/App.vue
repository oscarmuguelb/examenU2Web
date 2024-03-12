<template>
  <div class="container">
    <header>
      <div class="carousel">
        <b-carousel
          id="carousel-1"
          v-model="slide"
          :interval="4000"
          controls
          indicators
          background="#ababab"
          style="text-shadow: 1px 1px 2px #333"
          @sliding-start="onSlideStart"
          @sliding-end="onSlideEnd"
        >
          <b-carousel-slide
            v-for="(libro, index) in librosConImagenes"
            :key="index"
            :img-src="libro.imagen"
          >
          </b-carousel-slide>
        </b-carousel>
      </div>
    </header>
    <main>
      <div class="content">
        <h5>Ordenar por:</h5>
        <div class="buttons">
          <b-button-group>
            <b-button variant="info" @click="sortByAuthor">Autor</b-button>
            <b-button variant="info" @click="sortByDate">Fecha</b-button>
            <b-button variant="info" @click="toggleHideWithoutImage"
              >Ocultar sin imagen</b-button
            >
          </b-button-group>
          <br />
          <b-button pill variant="info" @click="modalShow = !modalShow"
            >Agregar libro</b-button
          >
          <div>
            <b-modal v-model="modalShow">
              <div class="FormLibro">
                <h1>Agregar libro</h1>
                <br />
                <b-form-input
                  placeholder="Nombre del libro"
                  v-model="bookName"
                ></b-form-input>
                <br />
                <b-form-input
                  placeholder="Autor"
                  v-model="author"
                ></b-form-input>
                <br />
                <label for="example-datepicker">Fecha de publicacion</label>
                <br />
                <b-form-datepicker
                  id="example-datepicker"
                  v-model="publicationDate"
                  class="mb-2"
                ></b-form-datepicker>
                <br />
                <b-form-file
                  id="file-small"
                  size="sm"
                  @change="onFileChange"
                ></b-form-file>
                <br />
                <b-button type="submit" variant="primary" @click="saveBook"
                  >Agregar</b-button
                >
              </div>
            </b-modal>
          </div>
        </div>
        <div class="libros">
          <b-card
            v-for="(libro, index) in librosToShow"
            :key="index"
            :title="libro.titulo"
            :img-src="libro.imagen"
            img-alt="Image"
            img-top
            tag="article"
            style="max-width: 20rem"
            class="mb-2"
            draggable="true"
            @dragstart="dragStart(index)"
          >
            <b-card-text>
              <b>Nombre del autor:</b> {{ libro.autor }}
              <b>Fecha de publicacion: </b> {{ libro.fechaPublicacion }}
            </b-card-text>
            <b-button variant="primary" @click="editCard(index)">Editar</b-button>
          </b-card>
        </div>
      </div>
      <div class="TZOI">
        <div class="edit-handle" @dragover.prevent @drop="editCard">
          <img
            src="src\components\img\edit.PNG"
            alt="delete"
            width="150"
            height="150"
          />
        </div>
        <div class="delete-handle" @dragover.prevent @drop="deleteCard">
          <img
            src="src\components\img\delete.PNG"
            alt="delete"
            width="150"
            height="150"
          />
        </div>
      </div>
      <b-modal v-model="editModalShow">
        <div class="FormLibro">
          <h1>Editar libro</h1>
          <br />
          <b-form-input
            placeholder="Nombre del libro"
            v-model="editBookName"
          ></b-form-input>
          <br />
          <b-form-input
            placeholder="Autor"
            v-model="editAuthor"
          ></b-form-input>
          <br />
          <label for="edit-datepicker">Fecha de publicacion</label>
          <br />
          <b-form-datepicker
            id="edit-datepicker"
            v-model="editPublicationDate"
            class="mb-2"
          ></b-form-datepicker>
          <br />
          <b-form-file
            id="edit-file-small"
            size="sm"
            @change="onEditFileChange"
          ></b-form-file>
          <br />
          <b-button type="submit" variant="primary" @click="saveEditedBook"
            >Guardar</b-button
          >
        </div>
      </b-modal>
    </main>
    <footer>
      <p>Examen Por Oscar Miguel Barrios Tecorral 8° C</p>
    </footer>
  </div>
</template>

<script>
export default {
  props: {},
  name: "App",
  data() {
    return {
      slide: 0,
      sliding: null,
      modalShow: false,
      editModalShow: false,
      bookName: "",
      author: "",
      publicationDate: null,
      editBookName: "",
      editAuthor: "",
      editPublicationDate: null,
      libros: [],
      imageFile: null,
      editImageFile: null,
      hideWithoutImage: false,
      draggedIndex: null,
      editedIndex: null,
    };
  },
  computed: {
    librosConImagenes() {
      return this.libros.filter((libro) => libro.imagen);
    },
    librosToShow() {
      if (this.hideWithoutImage) {
        return this.libros.filter((libro) => libro.imagen);
      } else {
        return this.libros;
      }
    },
  },
  methods: {
    onSlideStart(slide) {
      this.sliding = true;
    },
    onSlideEnd(slide) {
      this.sliding = false;
    },
    onFileChange(event) {
      this.imageFile = event.target.files[0];
    },
    onEditFileChange(event) {
      this.editImageFile = event.target.files[0];
    },
    async saveBook() {
      const randomNumber = Math.floor(Math.random() * 100) + 1;
      const id = randomNumber + Date.now().toString();
      const book = {
        id: id,
        titulo: this.bookName,
        autor: this.author,
        fechaPublicacion: this.publicationDate,
        imagen: this.imageFile
          ? await this.convertImageToBase64(this.imageFile)
          : "",
      };

      let savedBooks = localStorage.getItem("books");
      savedBooks = savedBooks ? JSON.parse(savedBooks) : [];

      savedBooks.push(book);

      localStorage.setItem("books", JSON.stringify(savedBooks));

      this.libros = savedBooks;

      this.bookName = "";
      this.author = "";
      this.publicationDate = null;
      this.imageFile = null;

      this.modalShow = false;
    },
    async saveEditedBook() {
      const editedBook = {
        id: this.libros[this.editedIndex].id,
        titulo: this.editBookName,
        autor: this.editAuthor,
        fechaPublicacion: this.editPublicationDate,
        imagen: this.editImageFile
          ? await this.convertImageToBase64(this.editImageFile)
          : this.libros[this.editedIndex].imagen,
      };

      this.libros.splice(this.editedIndex, 1, editedBook);

      localStorage.setItem("books", JSON.stringify(this.libros));

      this.editModalShow = false;
    },
    convertImageToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => {
          resolve(reader.result);
        };
        reader.onerror = (error) => reject(error);
      });
    },
    sortByAuthor() {
      this.libros.sort((a, b) => a.autor.localeCompare(b.autor));
    },
    sortByDate() {
      this.libros.sort(
        (a, b) => new Date(a.fechaPublicacion) - new Date(b.fechaPublicacion)
      );
    },
    toggleHideWithoutImage() {
      this.hideWithoutImage = !this.hideWithoutImage;
    },
    dragStart(index) {
      this.draggedIndex = index;
    },
    deleteCard() {
      this.libros.splice(this.draggedIndex, 1);

      localStorage.setItem("books", JSON.stringify(this.libros));
    },
    editCard(index) {
      this.editedIndex = index;
      this.editBookName = this.libros[index].titulo;
      this.editAuthor = this.libros[index].autor;
      this.editPublicationDate = this.libros[index].fechaPublicacion;
      this.editModalShow = true;
    },
  },
  created() {
    const savedBooks = localStorage.getItem("books");
    if (savedBooks) {
      this.libros = JSON.parse(savedBooks);
    }
  },
};
</script>

<style scoped>
.carousel {
  margin-bottom: 20px;
  max-height: 480px;
  width: 1024px;
  overflow: hidden;
}

.carousel img {
  height: auto;
  width: 100%;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.buttons {
  margin-top: 20px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.TZOI {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  margin-bottom: 20px;
  position: fixed;
  right: 0;
  top: 0;
}

.FormLibro {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgb(255, 255, 255);
}
.FormLibro b-form-input:first-child {
  margin-bottom: 0;
}

.libros {
  display: flex;
  display: flex;
  flex-wrap: wrap;
}
</style>

<style scoped>
.carousel {
  margin-bottom: 20px;
  max-height: 480px;
  width: 1024px;
  overflow: hidden;
}

.carousel img {
  height: auto;
  width: 100%;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.buttons {
  margin-top: 20px;
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.TZOI {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  margin-bottom: 20px;
  position: fixed;
  right: 0;
  top: 0;
}

.FormLibro {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgb(255, 255, 255);
}
.FormLibro b-form-input:first-child {
  margin-bottom: 0;
}

.libros {
  display: flex;
  display: flex;
  flex-wrap: wrap;
}
</style>
