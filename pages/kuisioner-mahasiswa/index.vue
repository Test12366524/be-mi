<script setup lang="ts">
import { ref } from 'vue';
import { VCol } from "vuetify/lib/components/index.mjs";

const { snackbar, confirmDialog } = useCommonStore();

const tableRef = ref();
const jadwal = ref();
const filter_semester = ref();
const filter_periode = ref();

useApi("jadwal-mata-kuliah/all").then(({ data }) => {
  jadwal.value = data;
});

useApi("master/semester/all/0").then(({ data }) => {
  filter_semester.value = data;
});

useApi("master/periode/all/0").then(({ data }) => {
  filter_periode.value = data;
});

const role_id = ref();
const status_action = ref();
const { user } = useAuthStore();

onMounted(() => {
//   useApi(`level/kuisioner/${user.role_id}`).then(({ data }) => {
//     if(data == 0){
//       navigateTo(`/not-authorized`);
//     }
//   });
  role_id.value = user.role_id;
  status_action.value = user.role_id === 1 || user.role_id === 6
});


</script>

<template>

  <VRow>
    <VCol cols="12">
      <AppTable
        ref="tableRef"
        title="Data Kuisioner"
        path="kuisioner/mahasiswa"
        :with-actions="role_id === 1 || role_id === 6"
        :headers="[
          {
            title: 'Mata Kuliah',
            key: 'mata_kuliah_name',
            sortable: false,
          },
          {
            title: 'Dosen',
            key: 'dosen_name',
            sortable: false,
          },
          {
            title: 'Pertemuan Ke',
            key: 'pertemuan_ke',
            sortable: false,
          },
          {
            title: 'Judul',
            key: 'title',
            sortable: false,
          },
          {
            title: 'Semester',
            key: 'semester_name',
            sortable: false,
          },
          {
            title: 'Tahun Ajar',
            key: 'periode_name',
            sortable: false,
          },
        ]"
      >
        <template #actions="{ item, remove }">
          <div class="d-flex gap-1">
            <IconBtn
              label="Detail"
              size="small"
              @click="
                () => {
                  navigateTo(`/kuisioner-mahasiswa/${item.id}`);
                }
              "
            >
              <VIcon icon="ri-file-list-line" /> Kerjakan
            </IconBtn>
          </div>
        </template>
      </AppTable>
    </VCol>
  </VRow>
</template>
