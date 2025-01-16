<script setup lang="ts">
import { VCol, VTextField } from "vuetify/lib/components/index.mjs";

const { confirmDialog } = useCommonStore();

const dialogSave = ref();
const tableRef = ref();

const form = {
  name: "",
  nidn: "",
  nip: "",
  birth_date: "",
  place_of_birth: "",
  gender: "",
  email: "",
  phone: "",
  join_date: "",
  pendidikan_terakhir: "",
  address: "",
  photo: null,
};


const { user } = useAuthStore();
const role_id = ref();
const previewPhoto = ref("");

onMounted(() => {
  useApi(`level/dosen/${user.role_id}`).then(({ data }) => {
    if(data == 0){
      navigateTo(`/not-authorized`);
    }
  });
  role_id.value = user.role_id;
});
</script>

<template>
  <SaveFileDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isDetail }"
    ref="dialogSave"
    width="1000"
    path="master/dosen"
    title="Tambah Dosen"
    edit-title="Edit Dosen"
    detail-title="Detail Dosen"
    :request-form="form"
    :refresh-callback="tableRef.refresh"
  >
    <VRow>
      <VCol cols="12" md="3">
        <VCol cols="12" align-self="center">
          <VImg
            max-height="300"
            max-width="200"
            class="mb-3"
            rounded
            border
            :src="
              previewPhoto ||
              'https://placehold.jp/30/fff/555/300x150.png?text=Foto'
            "
          />
          <FileInput
            v-if="!isDetail"
            v-model="formData.photo"
            accept="image/*"
            label="Upload Foto"
            @change="(data) => (previewPhoto = data.previewImageUrl)"
          />
        </VCol>
      </VCol>
      <VCol cols="12" md="9">
        <VCard>
          <VCardTitle class="mb-2"> Data Dosen </VCardTitle>
          <VCardText>
            <VRow>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.nidn"
                  type="number"
                  :error-messages="validationErrors.nidn"
                  label="NIDN"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.nip"
                  :error-messages="validationErrors.nip"
                  label="NIP"
                  :readonly="isDetail"
                  type="number"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.name"
                  :error-messages="validationErrors.name"
                  label="Nama Dosen"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VLabel>Jenis Kelamin</VLabel>
                <VRadioGroup
                  v-model="formData.gender"
                  inline
                  :error-messages="validationErrors.gender"
                  :readonly="isDetail"
                >
                  <VRadio label="Laki-laki" value="L" />
                  <VRadio label="Perempuan" value="P" />
                </VRadioGroup>
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.birth_date"
                  type="date"
                  :error-messages="validationErrors.birth_date"
                  label="Tanggal Lahir"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.place_of_birth"
                  :error-messages="validationErrors.place_of_birth"
                  label="Tempat Lahir"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.email"
                  type="email"
                  :error-messages="validationErrors.email"
                  label="Email"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.phone"
                  :error-messages="validationErrors.phone"
                  label="No. Handphone"
                  type="number"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="12" md="6">
                <VTextField
                  v-model="formData.join_date"
                  type="date"
                  :error-messages="validationErrors.join_date"
                  label="Tanggal Bergabung"
                  :readonly="isDetail"
                />
              </VCol>
              <VCol cols="6">
                <VTextField
                  v-model="formData.address"
                  :error-messages="validationErrors.address"
                  label="Alamat"
                  :readonly="isDetail"
                />
              </VCol>
            </VRow>
          </VCardText>
        </VCard>
      </VCol>
    </VRow>
  </SaveFileDialog>

  <VRow>
    <VCol cols="12">
      <VCard>
        <VCardItem>
          <VRow>
            <VCol cols="12" md="6" class="d-flex align-center gap-4">
              <VBtn
                v-if="role_id == 1"
                color="primary"
                @click="
                  () => {
                    previewPhoto = null;
                    dialogSave.show();
                  }
                "
              >
                <VIcon end icon="ri-add-fill" />
                Tambah Data
              </VBtn>

              <ImportFileExcel
                @done="tableRef.refresh()"
                path="master/dosen/import-excel"
              ></ImportFileExcel>
            </VCol>
          </VRow>
        </VCardItem>
      </VCard>
    </VCol>

    <VCol cols="12">
      <AppTable
        ref="tableRef"
        title="Data Dosen"
        path="master/dosen"
        :with-actions="true"
        :headers="[
          {
            title: 'Nama',
            key: 'name',
            sortable: false,
          },
          {
            title: 'NIDN',
            key: 'nidn',
            sortable: false,
          },
          {
            title: 'NIP',
            key: 'nip',
            sortable: false,
          },
          {
            title: 'Email',
            key: 'email',
            sortable: false,
          },
          {
            title: 'No. Handphone',
            key: 'phone',
            sortable: false,
          },
          {
            title: 'Tanggal Masuk',
            key: 'join_date',
            sortable: false,
          },
        ]"
      >
        <template #actions="{ item, remove }">
          <div class="d-flex gap-1">
            <IconBtn
              size="small"
              title="Detail"
              @click="
                () => {
                  const payload = { ...item };
                  payload.birth_date = formatFullDate(
                    payload.birth_date
                  ).simpleDate;
                  if (payload.photo) previewPhoto = getFileUrl(payload.photo);
                  dialogSave.show(payload, true);
                }
              "
            >
              <VIcon icon="ri-eye-line" />
            </IconBtn>
            <IconBtn
              size="small"
              @click="
                () => {
                  const payload = { ...item };
                  payload.birth_date = formatFullDate(
                    payload.birth_date
                  ).simpleDate;
                  if (payload.photo) previewPhoto = getFileUrl(payload.photo);
                  dialogSave.show(payload);
                }
              "
            >
              <VIcon icon="ri-pencil-line" />
            </IconBtn>
            <IconBtn
              size="small"
              @click="
                confirmDialog.show({
                  title: 'Hapus Dosen',
                  message: `Anda yakin ingin menghapus Dosen ${
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
