<script setup lang="ts">
import { VTextField } from 'vuetify/lib/components/index.mjs';

const { confirmDialog } = useCommonStore();

const dialogSave = ref();

const tableRef = ref();

const form = ref({
    user_id: null, // dosenid
    kelas_id: null, // dosenid
    mata_kuliah_id: null, // dosenid
    pertemuan_ke: 0, // dosenid
    catatan: "", // dosenid
    file: null, // dosenid
});

const dosen = ref([]);
const kelas = ref([]);
const mata_kuliah = ref([]);


onMounted(() => {
  const { user } = useAuthStore();
//   useApi(`level/progres-kerja/${user.role_id}`).then(({ data }) => {
//     if(data == 0){
//       navigateTo(`/not-authorized`);
//     }
//   });

    useApi("user/all-dosen").then(({ data }) => {
        dosen.value = data;
    });

    useApi("master/kelas/all").then(({ data }) => {
        kelas.value = data;
    });

    useApi("master/mata-kuliah/all").then(({ data }) => {
        mata_kuliah.value = data;
    });
});
</script>

<template>
  <SaveFileDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isEditing }"
    ref="dialogSave"
    path="progres-kerja"
    title="Tambah Progress Kerja"
    edit-title="Edit Progress Kerja"
    :default-form="form"
    :request-form="form"
    :refresh-callback="tableRef.refresh"
    width="600"
  >
    <VCol cols="12">
      <VAutocomplete
        v-model="formData.user_id"
        label="Dosen"
        :error-messages="validationErrors.user_id"
        placeholder="Pilih Dosen"
        :items="dosen"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
      />
    </VCol>

    <VCol cols="12">
      <VAutocomplete
        v-model="formData.kelas_id"
        label="Kelas"
        :error-messages="validationErrors.kelas_id"
        placeholder="Pilih Kelas"
        :items="kelas"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
      />
    </VCol>

    <VCol cols="12">
      <VAutocomplete
        v-model="formData.mata_kuliah_id"
        label="Mata Kuliah"
        :error-messages="validationErrors.mata_kuliah_id"
        placeholder="Pilih Mata Kuliah"
        :items="mata_kuliah"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
      />
    </VCol>

    <VCol cols="12">
      <VTextField
        v-model="formData.pertemuan_ke"
        type="number"
        :error-messages="validationErrors.pertemuan_ke"
        label="Pertemuan Ke"
      />
    </VCol>

    <VCol cols="12">
      <VTextField
        v-model="formData.catatan"
        :error-messages="validationErrors.catatan"
        label="Catatan"
      />
    </VCol>
    <VCol cols="12">
        <FileInput
            v-model="formData.file"
            label="Upload File"
        />
    </VCol>
  </SaveFileDialog>

  <VRow>
    <VCol cols="12">
      <VCard>
        <VCardItem>
          <VBtn
            color="primary" 
            @click="
              () => {
                dialogSave.show();
              }
            "
          >
            <VIcon end icon="ri-add-fill" />
            Tambah Data
          </VBtn>
        </VCardItem>
      </VCard>
    </VCol>

    <VCol cols="12">
      <AppTable
        ref="tableRef"
        title="Data Progress Kerja"
        path="progres-kerja"
        :with-actions="true"
        :headers="[
          {
            title: 'Dosen',
            key: 'user_name',
            sortable: false,
          },
          {
            title: 'Kelas',
            key: 'kelas_name',
            sortable: false,
          },
          {
            title: 'Mata Kuliah',
            key: 'mata_kuliah_name',
            sortable: false,
          },
          {
            title: 'Catatan',
            key: 'catatan',
            sortable: false,
          },
        ]"
      >
        <template #actions="{ item, remove }">
          <div class="d-flex gap-1">
            <IconBtn
              label="Edit"
              size="small"
              @click="
                () => {
                  const payload = { ...item };
                  dialogSave.show(payload, false);
                }
              "
            >
              <VIcon icon="ri-pencil-line" />
            </IconBtn>
            <IconBtn
              label="Hapus"
              size="small"
              @click="
                confirmDialog.show({
                  title: 'Hapus Progress Kerja',
                  message: `Anda yakin ingin menghapus Progress Kerja ${
                    (item as any).name
                  }?`,
                  onConfirm: () => remove((item as any).id),
                })
              "
            >
              <VIcon icon="ri-delete-bin-line" />
            </IconBtn>
          </div>
        </template>
      </AppTable>
    </VCol>
  </VRow>
</template>
