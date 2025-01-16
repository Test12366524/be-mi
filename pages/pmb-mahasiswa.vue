<script setup lang="ts">
import { VCol, VTextField } from 'vuetify/lib/components/index.mjs';

const { confirmDialog } = useCommonStore()

const dialogSave = ref()
const tableRef = ref()

const periode = ref();
const filter_periode = ref();

const jurusans = ref();
const angkatans = ref();

const form = ref({
  user_id: undefined,
  periode_id: undefined,
  jurusan_id: undefined,
  kelas_id: undefined,
  name: "",
  photo: "",
  fc_ijazah: "",
  surat_rekomendasi: "",
  fc_ktp: "",
  birth_date: "",
  place_of_birth: "",
  gender: "",
  religion: "",
  nik: "",
  entrance_date: "",
  email: "",
  phone: "",
  address: "",
  created_at: "",
  updated_at: "",
  province_id: "",
  city_id: "",
  district_id: "",
  subdistrict_id: "",
  post_code: "",
  pendidikan_terakhir: "",
  pengalaman_kursus: "",
  pengalaman_organisasi: "",
});

const religions = ref([
  { id: "Islam", text: "Islam" },
  { id: "Katolik", text: "Katolik" },
  { id: "Protestan", text: "Protestan" },
  { id: "Hindu", text: "Hindu" },
  { id: "Budha", text: "Budha" },
  { id: "Konghucu", text: "Konghucu" },
]);

const genders = ref([
  { id: "L", text: "Laki-laki" },
  { id: "P", text: "Perempuan" },
]);

const semester_id = ref<number | null>(null);
const periode_id = ref<number | null>(null);
const gender = ref("");

useApi("master/periode/all/1").then(({ data }) => {
  periode.value = data;
});

useApi("master/periode/all/0").then(({ data }) => {
  filter_periode.value = data;
});

useApi("master/jurusan/all").then(({ data }) => {
  jurusans.value = data;
});


const role_id = ref();
const status_action = ref();
const provinceList = ref([]);
const cityList = ref([]);
const districtList = ref([]);
const subDistrictList = ref([]);
const previewPhoto = ref(null);
const getProvinceList = async () => {
  const { data } = await useApi("province/all", {
    withLoader: false,
  });

  provinceList.value = data;
  cityList.value = [];
  districtList.value = [];
  subDistrictList.value = [];

  const resetKeys = ["city_id", "district_id", "subdistrict_id"];

  // dialogSave.value.resetData(form.value, resetKeys);
};

const getCityList = async (province_id: number) => {
  const { data } = await useApi(`city/all/${province_id}`, {
    withLoader: false,
  });

  cityList.value = data;
  districtList.value = [];
  subDistrictList.value = [];

  const resetKeys = ["district_id", "subdistrict_id"];

  // dialogSave.value.resetData(form.value, resetKeys);
};

const getDistrictList = async (city_id: number) => {
  const { data } = await useApi(`district/all/${city_id}`, {
    withLoader: false,
  });

  districtList.value = data;
  subDistrictList.value = [];

  const resetKeys = ["subdistrict_id"];

  // dialogSave.value.resetData(form.value, resetKeys);
};

const getSubDistrictList = async (district_id: number) => {
  const { data } = await useApi(`subdistrict/all/${district_id}`, {
    withLoader: false,
  });

  subDistrictList.value = data;
};

const handleAction = async (status, type, data) => {
  const url = `ppdb/change-status/${type}/${status}/${data.id}`

  const { errors, success } = await useApi(url, {
    withNotif: true,
    method: 'PUT',
    data,
    headers: {
      'Content-Type': 'multipart/form-data',
    },
  })

  if (success)
    tableRef.value.refresh()
}

const { user } = useAuthStore()

onMounted(() => {
  const { user } = useAuthStore();
  useApi(`level/pmb-mahasiswa/${user.role_id}`).then(({ data }) => {
    if(data == 0){
      navigateTo(`/not-authorized`);
    }
  });
  getProvinceList();
});
</script>

<template>
  <SaveFileDialog
    v-if="tableRef"
    v-slot="{ formData, validationErrors, isDetail }"
    ref="dialogSave"
    width="1200"
    path="ppdb"
    title="Tambah PMB Mahasiswa"
    edit-title="Edit PMB Mahasiswa"
    detail-title="Detail PMB Mahasiswa"
    :default-form="form"
    :refresh-callback="tableRef.refresh"
  >
    <VCol cols="12" md="3">
      <VCol cols="12">
        <VImg
          class="mb-3"
          rounded
          border
          max-height="300"
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
        <VCardTitle class="mb-2"> Data Pribadi </VCardTitle>
        <VCardText>
          <VRow>
            <VCol cols="12" md="6">
              <VTextField
                v-model="formData.name"
                :error-messages="validationErrors.name"
                label="Nama"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VTextField
                v-model="formData.nik"
                :error-messages="validationErrors.nik"
                label="NIK"
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
                v-model="formData.birth_date"
                type="date"
                :error-messages="validationErrors.birth_date"
                label="Tanggal Lahir"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.periode_id"
                label="Periode"
                :error-messages="validationErrors.periode_id"
                placeholder="Pilih Periode"
                :items="periode"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.jurusan_id"
                label="Jurusan"
                :error-messages="validationErrors.jurusan_id"
                placeholder="Pilih Jurusan"
                :items="jurusans"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :readonly="isDetail"
              />
            </VCol>
          </VRow>
        </VCardText>
      </VCard>

      <VCard class="mt-4">
        <VCardTitle class="mb-2"> Alamat Domisili </VCardTitle>
        <VCardText>
          <VRow>
            <VCol cols="12" md="6">
              <VTextField
                v-model="formData.address"
                :error-messages="validationErrors.address"
                label="Jalan"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.province_id"
                label="Provinsi"
                :error-messages="validationErrors.province_id"
                placeholder="Pilih Provinsi"
                :items="provinceList"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :readonly="isDetail"
                @update:model-value="getCityList"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.city_id"
                label="Kota/Kabupaten"
                :error-messages="validationErrors.city_id"
                placeholder="Pilih Kota/Kabupaten"
                :items="cityList"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :disabled="!formData.province_id"
                :readonly="isDetail"
                @update:model-value="getDistrictList"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.district_id"
                label="Kecamatan"
                :error-messages="validationErrors.district_id"
                placeholder="Pilih Kecamatan"
                :items="districtList"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :disabled="!formData.city_id"
                :readonly="isDetail"
                @update:model-value="getSubDistrictList"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VAutocomplete
                v-model="formData.subdistrict_id"
                label="Desa/Kelurahan"
                :error-messages="validationErrors.subdistrict_id"
                placeholder="Pilih Desa/Kelurahan"
                :items="subDistrictList"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
                :disabled="!formData.district_id"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="6">
              <VTextField
                v-model="formData.post_code"
                :error-messages="validationErrors.post_code"
                label="Kode Pos"
                :readonly="isDetail"
              />
            </VCol>
          </VRow>
        </VCardText>
      </VCard>

      <VCard class="mt-4">
        <VCardTitle class="mb-2"> Informasi Lainnya </VCardTitle>
        <VCardText>
          <VRow>
            <VCol cols="12" md="4">
              <VAutocomplete
                v-model="formData.religion"
                label="Agama"
                :error-messages="validationErrors.religion"
                placeholder="Pilih Agama"
                :items="religions"
                item-title="text"
                item-value="text"
                required
                clearable
                clear-icon="ri-close-line"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="4">
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
            <VCol cols="12" md="4">
              <VTextField
                v-model="formData.pendidikan_terakhir"
                :error-messages="validationErrors.pendidikan_terakhir"
                label="Pendidikan Terakhir"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="4">
              <VTextField
                v-model="formData.pengalaman_kursus"
                :error-messages="validationErrors.pengalaman_kursus"
                label="Pengalaman Kursus"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="4">
              <VTextField
                v-model="formData.pengalaman_organisasi"
                :error-messages="validationErrors.pengalaman_organisasi"
                label="Pengalaman Organisasi"
                :readonly="isDetail"
              />
            </VCol>
            <VCol cols="12" md="4">
              <VTextField
                v-model="formData.entrance_date"
                type="date"
                :error-messages="validationErrors.entrance_date"
                label="Tanggal Daftar"
                :readonly="isDetail"
              />
            </VCol>
          </VRow>
        </VCardText>
      </VCard>

      <VCard class="mt-4">
        <VCardTitle class="mb-2"> Data Berkas </VCardTitle>
        <VCardText>
          <VRow>
            <VCol cols="12" md="4">
              <FileInput
                v-if="!isDetail"
                v-model="formData.fc_ijazah"
                accept="application/pdf"
                label="Foto Copy Ijazah"
              />
            </VCol>
            <VCol cols="12" md="4">
              <FileInput
                v-if="!isDetail"
                v-model="formData.surat_rekomendasi"
                accept="application/pdf"
                label="Surat Rekomendasi"
              />
            </VCol>
            <VCol cols="12" md="4">
              <FileInput
                v-if="!isDetail"
                v-model="formData.fc_ktp"
                accept="application/pdf"
                label="Foto Copy KTP"
              />
            </VCol>
          </VRow>
        </VCardText>
      </VCard>

      <VCard class="mt-4">
        <VCardTitle class="mb-2"> Kontak dan Email </VCardTitle>
        <VCardText>
          <VRow>
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
          </VRow>
        </VCardText>
      </VCard>
    </VCol>
  </SaveFileDialog>

  <VRow>
    <VCol cols="12">
      <VCard>
        <VCardItem>
          <VRow>
            <VCol cols="12" md="6">
              <VBtn
                color="primary"
                @click="dialogSave.show()"
              >
                <VIcon
                  end
                  icon="ri-add-fill"
                />
                Tambah Data
              </VBtn>
            </VCol>
            <VCol cols="12" md="2" style="margin-block-start: 5px">
            </VCol>
            <VCol cols="12" md="2" style="margin-block-start: 5px">
              <VAutocomplete
                v-model="periode_id"
                label="Tahun Ajaran"
                placeholder="Pilih Tahun Ajaran"
                :items="filter_periode"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
              />
            </VCol>
            <VCol cols="12" md="2" style="margin-block-start: 5px">
              <VAutocomplete
                v-model="semester_id"
                label="Semester"
                placeholder="Pilih Semester"
                :items="filter_semester"
                item-title="text"
                item-value="id"
                required
                clearable
                clear-icon="ri-close-line"
              />
            </VCol>
          </VRow>
        </VCardItem>
      </VCard>
    </VCol>

    <VCol cols="12">
      <AppTable
        ref="tableRef"
        title="Data PMB Mahasiswa"
        path="ppdb"
        :with-actions="true"
        :periode_id="periode_id"
        :semester_id="semester_id"
        :headers="[
          {
            title: 'Kode',
            key: 'registration_code',
            sortable: false,
          },
          {
            title: 'Nama',
            key: 'name',
            sortable: false,
          },
          {
            title: 'NIK',
            key: 'nik',
            sortable: false,
          },
          {
            title: 'Jenis Kelamin',
            key: 'gender',
            sortable: false,
          },
          {
            title: 'No. Handphone',
            key: 'phone',
            sortable: false,
          },
          // {
          //   title: 'Pendidikan Terakhir',
          //   key: 'pendidikan_terakhir',
          //   sortable: false,
          // },
          {
            title: 'Status',
            key: 'status_all',
            sortable: false,
          },
        ]"
      >
        <template #actions="{ item, remove }">
          <div class="d-flex gap-1">
            <VBtn
              v-if="item.tripay_status_transaction == 0"
              title="Pembayaran diterima"
              size="x-small"
              style="border-radius: 0.375rem !important;"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Approve Payment',
                    message: `Anda yakin ingin mengubah status jadi approve?`,
                    onConfirm: () => handleAction('payment', 'approved', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-checkbox-circle-line"
              />
              Pembayaran
            </VBtn>
            <VBtn
              v-if="item.tripay_status_transaction == 0"
              size="x-small"
              title="Pembayaran ditolak"
              style="border-radius: 0.375rem !important;"
              color="secondary"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Reject Payment',
                    message: `Anda yakin ingin mengubah status jadi reject?`,
                    onConfirm: () => handleAction('payment', 'rejected', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-subtract-line"
              />
              Pembayaran
            </VBtn>
            <VBtn
              v-if="item.status == 0 && item.tripay_status_transaction == 1"
              size="x-small"
              style="border-radius: 0.375rem !important;"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Approve Verification',
                    message: `Anda yakin ingin mengubah status jadi approve?`,
                    onConfirm: () => handleAction('data', 'approved', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-checkbox-circle-line"
              />
              Verifikasi Data Diterima
            </VBtn>
            <VBtn
              v-if="item.status == 0 && item.tripay_status_transaction == 1"
              size="x-small"
              style="border-radius: 0.375rem !important;"
              color="secondary"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Reject Verification',
                    message: `Anda yakin ingin mengubah status jadi reject?`,
                    onConfirm: () => handleAction('data', 'rejected', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-subtract-line"
              />
              Verifikasi Data Ditolak
            </VBtn>
            <VBtn
              v-if="item.status_test == 0 && item.status == 1"
              size="x-small"
              style="border-radius: 0.375rem !important;"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Status Lulus',
                    message: `Anda yakin ingin mengubah status jadi lulus?`,
                    onConfirm: () => handleAction('test', 'approved', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-checkbox-circle-line"
              />
              Lulus
            </VBtn>
            <VBtn
              v-if="item.status_test == 0 && item.status == 1"
              size="x-small"
              style="border-radius: 0.375rem !important;"
              color="secondary"
              @click="
                () => {
                  confirmDialog.show({
                    title: 'Status Tidak Lulus',
                    message: `Anda yakin ingin mengubah status jadi tidak lulus?`,
                    onConfirm: () => handleAction('test', 'rejected', item),
                  });
                }
              "
            >
              <VIcon
                start
                icon="ri-subtract-line"
              />
              Tidak Lulus
            </VBtn>
            <IconBtn
              size="small"
              title="Detail"
              @click="
                () => {
                  const payload = { ...item };
                  payload.mariage_status = String(item.mariage_status);
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
                  payload.mariage_status = String(item.mariage_status);
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
                  title: 'Hapus PMB Mahasiswa',
                  message: `Anda yakin ingin menghapus PMB Mahasiswa ${
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
