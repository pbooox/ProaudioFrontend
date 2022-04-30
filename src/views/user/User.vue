<template>
  <div>
    <!-- Modal desactivar -->
    <b-modal id="modal-des" title="Desactivar">
      <p class="my-4">
        ¿Desea desactivar el tipo de medicamento: {{ form.name }} ?
      </p>
      <template #modal-footer="{}">
        <b-button
          size="sm"
          variant="success"
          @click="
            onState();
            $bvModal.hide('modal-des');
          "
        >
          Desactivar
        </b-button>
        <b-button
          size="sm"
          variant="danger"
          @click="$bvModal.hide('modal-des')"
        >
          Cancelar
        </b-button>
      </template>
    </b-modal>
    <!-- Modal activar -->
    <b-modal id="modal-react" title="Activar">
      <p class="my-4">
        ¿Desea activar el tipo de medicamento: {{ form.name }} ?
      </p>
      <template #modal-footer="{}">
        <b-button
          size="sm"
          variant="success"
          @click="
            onState();
            $bvModal.hide('modal-react');
          "
        >
          Activar
        </b-button>
        <b-button
          size="sm"
          variant="danger"
          @click="$bvModal.hide('modal-react')"
        >
          Cancelar
        </b-button>
      </template>
    </b-modal>
    <!-- Modal editar -->
    <b-modal
      no-close-on-backdrop
      ref="modaled"
      id="modal-edit"
      scrollable
      title="Editar tipo de medicamento"
    >
      <b-form>
        <b-form-group label="Nombre:">
          <b-form-input
            v-model.trim="$v.form.name.$model"
            :state="!$v.form.name.$error"
            placeholder="Ingresar nombre de tipo de medicamento"
          ></b-form-input>
          <b-form-invalid-feedback
            >Debe ingresar el nombre</b-form-invalid-feedback
          >
        </b-form-group>
        <!-- 
        <b-form-group label="Descripción:">
          <b-form-input
            v-model="form.description"
            placeholder="Ingresar descripción"
          ></b-form-input>
        </b-form-group> -->
      </b-form>
      <template #modal-footer="{}">
        <b-button type="submit" variant="primary" @click="onValidate('update')"
          >Actualizar</b-button
        >
        <b-button @click="$bvModal.hide('modal-edit')" variant="danger"
          >Cancelar</b-button
        >
      </template>
    </b-modal>
    <!-- Modal nuevo registro -->
    <b-modal
      no-close-on-backdrop
      @show="openModal('save')"
      @hidden="openModal('save')"
      id="modalSave"
      ref="modalSave"
      title="Registro tipo de medicamento"
    >
      <b-form>
        <b-form-group label="Nombre:">
          <b-form-input
            v-model.trim="$v.form.name.$model"
            :state="!$v.form.name.$error"
            placeholder="Ingresar nombre de tipo de medicamento"
          ></b-form-input>
          <b-form-invalid-feedback
            >Debe ingresar el nombre</b-form-invalid-feedback
          >
        </b-form-group>

        <!-- <b-form-group label="Descripción:">
          <b-form-input
            v-model="form.description"
            placeholder="Ingresar descripción"
          ></b-form-input>
        </b-form-group> -->
      </b-form>
      <template #modal-footer="{}">
        <b-button type="submit" variant="primary" @click="onValidate('save')"
          >Guardar</b-button
        >
        <b-button variant="danger" @click="closeModal('save')"
          >Cancelar</b-button
        >
      </template>
    </b-modal>
    <!-- Heading -->
    <datatable-heading
      :title="'Tipo de medicamento'"
      :changePageSize="changePageSize"
      :searchChange="searchChange"
      :from="from"
      :to="to"
      :total="total"
      :perPage="perPage"
    ></datatable-heading>
    <!-- Tabla general -->
    <b-row>
      <b-colxx xxs="12">
        <!-- Contenido -->
        <vuetable
          ref="vuetable"
          class="table-divided order-with-arrow"
          :api-url="apiBase"
          :query-params="makeQueryParams"
          :per-page="perPage"
          :reactive-api-url="true"
          :fields="fields"
          pagination-path
          @vuetable:pagination-data="onPaginationData"
        >
          <!-- Estado -->
          <template slot="estado" slot-scope="props">
            <h5 v-if="props.rowData.estado == 1">
              <b-badge variant="light"
                ><h6 class="success"><strong>ACTIVO</strong></h6></b-badge
              >
            </h5>
            <h5 v-else>
              <b-badge variant="light"
                ><h6 class="danger"><strong>INACTIVO</strong></h6></b-badge
              >
            </h5>
          </template>

          <!-- Botones -->
          <template slot="actions" slot-scope="props">
            <b-button-group>
              <b-button
                @click="setData(props.rowData)"
                v-b-modal.modal-edit
                class="mb-2"
                size="sm"
                variant="outline-warning"
                ><i :class="'simple-icon-pencil'"
              /></b-button>
              <b-button
                @click="
                  setData(props.rowData);
                  props.rowData.estado == 1
                    ? $bvModal.show('modal-des')
                    : $bvModal.show('modal-react');
                "
                class="mb-2"
                size="sm"
                :variant="
                  props.rowData.estado == 1 ? 'outline-danger' : 'outline-info'
                "
              >
                <i
                  :class="
                    props.rowData.estado == 1
                      ? 'simple-icon-trash'
                      : 'simple-icon-check'
                  "
              /></b-button>
            </b-button-group>
          </template>
        </vuetable>
        <!-- Paginacion -->
        <vuetable-pagination-bootstrap
          class="mt-4"
          ref="pagination"
          @vuetable-pagination:change-page="onChangePage"
        />
      </b-colxx>
    </b-row>
  </div>
</template>
<script>
import Vuetable from "vuetable-2/src/components/Vuetable";
import VuetablePaginationBootstrap from "../../components/Common/VuetablePaginationBootstrap";
import { apiUrl } from "../../constants/config";
import DatatableHeading from "../../containers/datatable/DatatableHeading";
import axios from "axios";
import { validationMixin } from "vuelidate";
const { required } = require("vuelidate/lib/validators");

export default {
  props: ["title"],
  components: {
    vuetable: Vuetable,
    "vuetable-pagination-bootstrap": VuetablePaginationBootstrap,
    "datatable-heading": DatatableHeading,
  },
  data() {
    return {
      isLoad: false,
      apiBase: apiUrl + "/tipo_medicamento/list",
      page: 1,
      perPage: 5,
      form: {
        name: "",
        description: "",
        state: 1,
        id: 0,
      },
      search: "",
      from: 0,
      to: 0,
      total: 0,
      lastPage: 0,
      /* Nombres de los datos de la tabla */
      fields: [
        {
          name: "__slot:actions",
          title: "Acciones",
          titleClass: "",
          dataClass: "text-muted",
          width: "15%",
        },
        {
          name: "nombre",
          sortField: "nombre",
          title: "Nombre",
          titleClass: "",
          dataClass: "list-item-heading",
          width: "25%",
        },
        // {
        //   name: "descripcion",
        //   sortField: "descripcion",
        //   title: "Descripción",
        //   titleClass: "",
        //   dataClass: "list-item-heading",
        //   width: "50%",
        // },
        {
          name: "__slot:estado",
          title: "Estado",
          titleClass: "",
          dataClass: "text-muted",
          width: "25%",
        },
      ],
    };
  },
  mixins: [validationMixin],
  validations: {
    form: {
      name: {
        required,
      },
    },
  },
  methods: {
    openModal(model, action) {
      switch (model) {
        case "save": {
          this.$v.$reset();
          this.form.id = 0;
          this.form.name = "";
          // this.form.description = "";
          this.form.state = 1;
          break;
        }
      }
    },
    closeModal(model, action) {
      switch (model) {
        case "save": {
          this.$v.$reset();
          this.$refs["modalSave"].hide();
          this.form.name = "";
          // this.form.description = "";
          this.form.state = 1;
          break;
        }
      }
    },
    /* Guardar */
    onSave() {
      const me = this;
      this.$refs["modalSave"].hide();
      axios
        .post(apiUrl + "/tipo_medicamento/create", {
          nombre: this.form.name,
        })
        .then((response) => {
          me.$notify(
            "primary filled",
            "Guardado",
            "Tipo medicamento " + this.form.name + " guardado",
            { duration: 3000, permanent: false }
          );
          me.$refs.vuetable.refresh();
          me.form.name = "";
          me.form.description = "";
        })
        .catch((error) => {
          this.errorMessage = error.message;
          console.error("Error!", error);
        });
    },
    /* Actualizar */
    onUpdate() {
      const me = this;
      this.$refs.modaled.hide();
      axios
        .put(apiUrl + "/tipo_medicamento/update", {
          id: this.form.id,
          nombre: this.form.name,
        })
        .then((response) => {
          me.$notify(
            "secondary filled",
            "Actualizado",
            "Tipo medicamento " + this.form.name + " actualizado",
            { duration: 3000, permanent: false }
          );
          me.$refs.vuetable.refresh();
          me.form.name = "";
        })
        .catch((error) => {
          this.errorMessage = error.message;
          console.error("Error!", error);
        });
    },
    // Estados
    onState() {
      const me = this;
      if (this.form.state == 1) {
        axios
          .put(apiUrl + "/tipo_medicamento/deactivate", {
            id: this.form.id,
          })
          .then((response) => {
            me.$notify(
              "warning",
              "Desactivado",
              "Tipo medicamento " + this.form.name + " desactivado",
              { duration: 3000, permanent: false }
            );
            me.$refs.vuetable.refresh();
          })
          .catch((error) => {
            this.errorMessage = error.message;
            console.error("There was an error!", error);
          });
      } else {
        axios
          .put(apiUrl + "/tipo_medicamento/activate", {
            id: this.form.id,
          })
          .then((response) => {
            me.$notify(
              "success",
              "Activado",
              "Tipo medicamento " + this.form.name + " activado",
              { duration: 3000, permanent: false }
            );
            me.$refs.vuetable.refresh();
          })
          .catch((error) => {
            this.errorMessage = error.message;
            console.error("There was an error!", error);
          });
      }
    },
    // Validar
    onValidate(action) {
      this.$v.$touch();
      if (this.form.name != "") {
        if (action == "save") {
          this.onSave();
        } else if (action == "update") {
          this.onUpdate();
        }
      }
    },
    // Setear datos en el modal
    setData(data) {
      console.log(data);
      this.form.name = data.nombre;
      // this.form.description = data.descripcion;
      this.form.state = data.estado;
      this.form.id = data.id;
    },

    searchChange(val) {
      this.search = val.toLowerCase();
      this.$refs.vuetable.refresh();
    },
    makeQueryParams(sortOrder, currentPage, perPage) {
      return sortOrder[0]
        ? {
            criterio: sortOrder[0] ? sortOrder[0].sortField : "createdAt",
            order: sortOrder[0] ? sortOrder[0].direction : "desc",
            page: currentPage,
            limit: this.perPage,
            search: this.search,
          }
        : {
            criterio: sortOrder[0] ? sortOrder[0].sortField : "createdAt",
            order: sortOrder[0] ? sortOrder[0].direction : "desc",
            page: currentPage,
            limit: this.perPage,
            search: this.search,
          };
    },
    onPaginationData(paginationData) {
      this.from = paginationData.from;
      this.to = paginationData.to;
      this.total = paginationData.total;
      this.lastPage = paginationData.last_page;
      this.items = paginationData.data;
      this.$refs.pagination.setPaginationData(paginationData);
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page);
    },
    changePageSize(perPage) {
      this.perPage = perPage;
      this.$refs.vuetable.refresh();
    },
  },
};
</script>

