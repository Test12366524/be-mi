<script setup lang="ts">
const { confirmDialog } = useCommonStore();

const dialogSave = ref();

const tableRef = ref();

const form = ref({
  dosen_id: null, // dosenid
  mata_kuliah_id: null,
  biaya: 0,
});

const teacherList = ref([]);
const mataKuliahList = ref([]);

const getAllTeacher = async () => {
  useApi("master/dosen/all").then(({ data }) => {
    teacherList.value = data;
  });
};

const getAllMataKuliah = async () => {
  useApi("master/mata-kuliah/all").then(({ data }) => {
    mataKuliahList.value = data;
  });
};

onMounted(() => {
  const { user } = useAuthStore();
  useApi(`level/dosen-bimbel/${user.role_id}`).then(({ data }) => {
    if(data == 0){
      navigateTo(`/not-authorized`);
    }
  });
  getAllTeacher();
  getAllMataKuliah();
});
</script>

<template>
  <SaveDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isEditing }"
    ref="dialogSave"
    path="dosen-bimbel"
    title="Tambah Dosen Bimbel"
    edit-title="Edit Dosen Bimbel"
    :default-form="form"
    :request-form="form"
    :refresh-callback="tableRef.refresh"
    width="600"
  >
    <VCol cols="12">
      <VAutocomplete
        v-model="formData.dosen_id"
        label="Dosen"
        :error-messages="validationErrors.dosen_id"
        placeholder="Pilih Dosen"
        :items="teacherList"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
        :readonly="isDetail"
      />
    </VCol>
    <VCol cols="12">
      <VAutocomplete
        v-model="formData.mata_kuliah_id"
        label="Mata Kuliah"
        :error-messages="validationErrors.mata_kuliah_id"
        placeholder="Pilih Mata Kuliah"
        :items="mataKuliahList"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
        :readonly="isDetail"
      />
    </VCol>

    <VCol cols="12">
      <CurrencyInput
        v-model="formData.biaya"
        :error-messages="validationErrors.biaya"
        :readonly="isDetail"
        label="Biaya"
      />
    </VCol>
  </SaveDialog>

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
        title="Data Dosen Bimbel"
        path="dosen-bimbel"
        :with-actions="true"
        :headers="[
          {
            title: 'Dosen',
            key: 'dosen_name',
            sortable: false,
          },
          {
            title: 'Mata Kuliah',
            key: 'mata_kuliah_name',
            sortable: false,
          },
          {
            title: 'Biaya',
            key: 'biaya',
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
                  title: 'Hapus Surat Masuk',
                  message: `Anda yakin ingin menghapus Surat Masuk ${
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
