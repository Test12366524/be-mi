<script setup lang="ts">
import { useAuthStore } from "@/stores/auth"; // Sesuaikan dengan path store Anda
import { useCommonStore } from "@/stores/common"; // Sesuaikan dengan path store Anda
import { onMounted, ref } from "vue";

const { confirmDialog } = useCommonStore();
const dialogSave = ref();
const tableRef = ref();

const form = ref({
  mahasiswa_id: "",
  judul_skripsi: "",
  description: "",
  status: "",
});

const role_id = ref();
const mahasiswaList = ref([]);

const getMahasiswaList = () => {
  useApi("mahasiswa/all").then(({ data }) => {
    mahasiswaList.value = data;
  });
};

onMounted(() => {
  const { user } = useAuthStore();

  getMahasiswaList();
});

// Fungsi untuk membuka dialog dengan data yang ada
const openDialog = (item = null) => {
  if (item) {
    // Jika item ada, set form dengan data item
    form.value = {
      mahasiswa_id: item.mahasiswa_id,
      judul_skripsi: item.judul_skripsi,
      description: item.description,
      status: item.status,
    };
  } else {
    // Jika tidak ada item, reset form
    form.value = {
      mahasiswa_id: "",
      judul_skripsi: "",
      description: "",
      status: "",
    };
  }
  dialogSave.value.show();
};
</script>

<template>
  <SaveDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isEditing }"
    ref="dialogSave"
    path="master/fakultas"
    title="Tambah Pengajuan Skripsi"
    edit-title="Edit Pengajuan Skripsi"
    :default-form="form"
    :refresh-callback="tableRef.refresh"
  >
    <VCol cols="12">
      <VAutocomplete
        v-model="formData.mahasiswa_id"
        :error-messages="validationErrors.mahasiswa_id"
        label="Nama Mahasiswa"
        placeholder="Pilih Mahasiswa"
        :items="mahasiswaList"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
      />
    </VCol>

    <VCol cols="12">
      <VTextField
        v-model="formData.judul_skripsi"
        :error-messages="validationErrors.judul_skripsi"
        label="Judul Skripsi"
      />
    </VCol>

    <VCol cols="12">
      <VTextarea
        v-model="formData.description"
        :error-messages="validationErrors.description"
        label="Deskripsi"
      />
    </VCol>

    <VCol cols="12">
      <VLabel>Status</VLabel>
      <VRadioGroup
        v-model="formData.status"
        inline
        :error-messages="validationErrors.status"
      >
        <VRadio label="Terima" :value="0" />
        <VRadio label="Tolak" :value="1" />
      </VRadioGroup>
    </VCol>
  </SaveDialog>

  <VRow>
    <VCol cols="12">
      <VCard>
        <VCardItem>
          <VRow>
            <VCol>
              <VBtn color="primary" @click="openDialog">
                <VIcon end icon="ri-add-fill" />
                Tambah Data
              </VBtn>
            </VCol>
          </VRow>
        </VCardItem>
      </VCard>
    </VCol>

    <VCol cols="12">
      <AppTable
        ref="tableRef"
        title="Data Pengajuan Skripsi"
        path="master/fakultas"
        :with-actions="true"
        :headers="[
          {
            title: 'Nama Mahasiswa',
            key: 'name',
            sortable: false,
          },
          {
            title: 'Judul Skripsi',
            key: 'judul_skripsi',
            sortable: false,
          },
          {
            title: 'Deskripsi',
            key: 'description',
            sortable: false,
          },
          {
            title: 'Status',
            key: 'status',
            sortable: false,
          },
        ]"
      >
        <template #actions="{ item, remove }">
          <div class="d-flex gap-1">
            <IconBtn size="small" @click="openDialog(item)">
              <VIcon icon="ri-pencil-line" />
            </IconBtn>
            <IconBtn
              size="small"
              @click="
                confirmDialog.show({
                  title: 'Hapus data',
                  message: `Anda yakin ingin menghapus data ${
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
