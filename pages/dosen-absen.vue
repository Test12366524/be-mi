<script setup lang="ts">
const { confirmDialog } = useCommonStore();

const dialogSave = ref();

const tableRef = ref();
const { user } = useAuthStore();
const role_id = ref(null);
const form = ref({
  user_id: null,
  tanggal: "",
  jam_masuk: "",
  jam_keluar: "",
  kehadiran: "",
});
const isAttendanceIn = ref(true);

const teacherList = ref([]);

const getAllTeacher = async () => {
  useApi("user/all-dosen").then(({ data }) => {
    console.log(data);
    teacherList.value = data;
  });
};

onMounted(() => {
  const roleId = Number(user.role_id);
  console.log(roleId);
  if (roleId !== 1 && roleId !== 2) return navigateTo(`/not-authorized`);
  role_id.value = roleId;
  if (roleId === 2) {
    form.value.user_id = user.id;
    form.value.kehadiran = "Hadir";
  } else {
    getAllTeacher();
  }
});
</script>

<template>
  <SaveDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isEditing }"
    ref="dialogSave"
    path="absen-dosen"
    :title="role_id === 2 ? 'Absen Masuk' : 'Tambah Absen Dosen'"
    :edit-title="role_id === 2 ? 'Absen Keluar' : 'Edit Absen Dosen'"
    :default-form="form"
    :request-form="form"
    :refresh-callback="tableRef.refresh"
    :action-btn-text="role_id === 2 ? 'Submit' : null"
    width="600"
  >
    <VCol cols="12" v-if="role_id === 1">
      <VAutocomplete
        v-model="formData.user_id"
        label="Dosen"
        :error-messages="validationErrors.user_id"
        placeholder="Pilih Dosen"
        :items="teacherList"
        item-title="text"
        item-value="id"
        required
        clearable
        clear-icon="ri-close-line"
      />
    </VCol>

    <VCol cols="12" md="12">
      <VTextField
        v-model="formData.tanggal"
        type="date"
        :error-messages="validationErrors.tanggal"
        label="Tanggal"
        :readonly="role_id === 2"
      />
    </VCol>
    <VCol cols="12" md="12" v-if="role_id === 2 ? isAttendanceIn : true">
      <VTextField
        v-model="formData.jam_masuk"
        type="time"
        :error-messages="validationErrors.jam_masuk"
        label="Jam Masuk"
        :readonly="role_id === 2"
      />
    </VCol>

    <VCol cols="12" md="12" v-if="role_id === 2 ? !isAttendanceIn : true">
      <VTextField
        v-model="formData.jam_keluar"
        type="time"
        :error-messages="validationErrors.jam_keluar"
        label="Jam Keluar"
      />
    </VCol>

    <VCol cols="12" md="12" v-if="role_id === 1">
      <VLabel>Kehadiran</VLabel>
      <VRadioGroup
        v-model="formData.kehadiran"
        inline
        :error-messages="validationErrors.kehadiran"
      >
        <VRadio label="Hadir" value="Hadir" />
        <VRadio label="Izin" value="Izin" />
        <VRadio label="Sakit" value="Sakit" />
        <VRadio label="Alpa" value="Alpa" />
      </VRadioGroup>
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
                if (role_id === 2) {
                  isAttendanceIn = true;
                  form.tanggal = formatFullDate(new Date()).localDateNow;
                  console.log(form.tanggal);
                  form.jam_masuk = formatFullDate(new Date()).localTimeNow;
                  console.log(form.jam_masuk);
                }
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
        title="Data Absen Dosen"
        path="absen-dosen"
        :with-actions="true"
        :headers="[
          {
            title: 'Dosen',
            key: 'user_name',
            sortable: false,
          },
          {
            title: 'Tanggal',
            key: 'tanggal',
            sortable: false,
          },
          {
            title: 'Jam Masuk',
            key: 'jam_masuk',
            sortable: false,
          },
          {
            title: 'Jam Keluar',
            key: 'jam_keluar',
            sortable: false,
          },
          {
            title: 'Kehadiran',
            key: 'kehadiran',
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
                  payload.tanggal = formatFullDate(payload.tanggal).simpleDate;
                  if (role_id === 2) {
                    isAttendanceIn = false;

                    payload.jam_keluar = formatFullDate(
                      new Date()
                    ).localTimeNow;
                  }
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
