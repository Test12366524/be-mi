<script setup lang="ts">
import { useAuthStore } from "@/stores/auth"; // Sesuaikan dengan path store Anda
import { useCommonStore } from "@/stores/common"; // Sesuaikan dengan path store Anda
import { onMounted, ref } from "vue";

const { confirmDialog } = useCommonStore();
const dialogSave = ref();
const tableRef = ref();

const form = ref({
  mahasiswa_id: "",
  place: "",
  start_date: "",
  end_date: "",
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
      place: item.place,
      start_date: item.start_date,
      end_date: item.end_date,
      status: item.status,
    };
  } else {
    // Jika tidak ada item, reset form
    form.value = {
      mahasiswa_id: "",
      place: "",
      start_date: "",
      end_date: "",
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
    title="Tambah Pengajuan KKN"
    edit-title="Edit Pengajuan KKN"
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
        v-model="formData.place"
        :error-messages="validationErrors.place"
        label="Tempat"
      />
    </VCol>

    <VCol cols="12" md="4">
      <VTextField
        v-model="formData.start_date"
        :error-messages="validationErrors.start_date"
        label="Tanggal Mulai"
        type="date"
      />
    </VCol>

    <VCol cols="12" md="4">
      <VTextField
        v-model="formData.end_date"
        :error-messages="validationErrors.end_date"
        label="Tanggal Selesai"
        type="date"
      />
    </VCol>

    <VCol cols="12" md="4" style="margin-block-start: -0.5rem">
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
        title="Data Pengajuan KKN"
        path="master/fakultas"
        :with-actions="true"
        :headers="[
          {
            title: 'Nama Mahasiswa',
            key: 'name',
            sortable: false,
          },
          {
            title: 'Tempat',
            key: 'place',
            sortable: false,
          },
          {
            title: 'Nama Kelompok',
            key: 'group_name',
            sortable: false,
          },
          {
            title: 'Tanggal Mulai',
            key: 'start_date',
            sortable: false,
          },
          {
            title: 'Tanggal Selesai',
            key: 'end_data',
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
