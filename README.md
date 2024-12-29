```
Cách 1: Theo dõi thay đổi từ props.data bằng watch
Sử dụng watch để cập nhật registData mỗi khi props.data thay đổi.

Cập nhật script:

ts
Sao chép mã
const registData = ref({ ...props.data });

watch(
  () => props.data,
  (newValue) => {
    registData.value = { ...newValue };
  },
  { immediate: true }
);
Cách 2: Sử dụng trực tiếp props.data
Nếu không cần chỉnh sửa dữ liệu trong component con, bạn có thể dùng props.data trực tiếp.

Cập nhật template:

vue
Sao chép mã
<MTextField
  v-model="props.data[field.name]" <!-- Dùng trực tiếp props.data -->
  :label="field.label"
  :placeholder="field.placeholder"
  :error-message="touched[field.name] ? errors[field.name] : undefined"
  @blur="validateField(field.name)"
/>
Cách 3: Thay thế v-model bằng @input và sử dụng hai chiều
Nếu bạn cần cập nhật dữ liệu từ component con, emit sự kiện update:data để thông báo lên component cha.

Cập nhật script:

ts
Sao chép mã
const updateField = (name: string, value: string) => {
  registData.value[name] = value;
  emits('update:data', registData.value);
};
Cập nhật template:

vue
Sao chép mã
<MTextField
  :value="registData[field.name]"
  @input="value => updateField(field.name, value)"
  :label="field.label"
  :placeholder="field.placeholder"
  :error-message="touched[field.name] ? errors[field.name] : undefined"
  @blur="validateField(field.name)"
/>
Kết luận
Nếu chỉ cần hiển thị: Cách 2 là đơn giản nhất.
Nếu cần chỉnh sửa dữ liệu: Cách 1 hoặc Cách 3 giúp đảm bảo dữ liệu đồng bộ giữa component cha và con.
```
```
dieu chinh viet vo la hien thi luon
<script setup lang="ts">
import * as yup from "yup";
import MTextField from "~/components/mocules/MTextField/index.vue";
import { ref, reactive } from "vue";

type RegistData = Record<string, string>;

interface Field {
  name: string;
  label: string;
  placeholder: string;
  validation: yup.StringSchema<string>;
}

const emits = defineEmits<{
  (e: "regist"): void;
}>();

interface Props {
  fields: Field[];
  registData: RegistData;
}

class RegistController {
  public fields: Field[];
  public props;
  public registData: Ref<RegistData>;
  public errors = reactive<Record<string, string>>({});
  public touched = reactive<Record<string, boolean>>({}); // Thêm trạng thái `touched`

  constructor(props: Props) {
    this.fields = props.fields;
    this.props = props;
    this.registData = ref<RegistData>({ ...this.props.registData });
    this.resetErrors();

    // Ràng buộc các phương thức
    this.validateAll = this.validateAll.bind(this);
    this.regist = this.regist.bind(this);
  }

  async validateField(name: string) {
    const field = this.fields.find((f) => f.name === name);
    if (!field) return;
    this.touched[name] = true; // Đánh dấu trường đã bị "touch"
    try {
      await field.validation.validate(this.registData.value[name]);
      this.errors[name] = "";
    } catch (error: any) {
      this.errors[name] = error.message;
    }
  }

  async validateAll() {
    this.fields.forEach((field) => (this.touched[field.name] = true)); // Đánh dấu tất cả trường đã "touch"
    await Promise.all(this.fields.map((field) => this.validateField(field.name)));
    return Object.values(this.errors).every((error) => !error);
  }

  resetErrors() {
    this.fields.forEach((field) => {
      this.errors[field.name] = "";
      this.touched[field.name] = false; // Đặt lại `touched`
    });
  }

  async regist() {
    if (await this.validateAll()) {
      console.log("Data:", this.registData.value);
      emits("regist");
    } else {
      console.warn("Validation failed");
    }
  }
}

const props = defineProps<Props>();
const controller = new RegistController(props);

const { fields, registData, errors, validateField, touched, regist } = controller;
</script>

<template>
  <v-row>
    <v-col cols="12" md="6" v-for="field in fields" :key="field.name">
      <MTextField
        v-model="registData[field.name]"
        :label="field.label"
        :placeholder="field.placeholder"
        :error-message="touched[field.name] ? errors[field.name] : undefined"
        @blur="validateField(field.name)"
      />
    </v-col>
    <v-col cols="12" md="6">
      <v-btn @click="regist">Regist</v-btn>
    </v-col>
  </v-row>
</template>

```
```
parent
<template>
  <v-container>
    <v-btn @click="openUpdateForm(user)">Edit User</v-btn>

    <UpdateForm
      v-if="showForm"
      :data="selectedData"
      :fields="fields"
      @update="handleUpdate"
      @cancel="closeForm"
    />
  </v-container>
</template>

<script setup lang="ts">
import { ref } from "vue";
import UpdateForm from "@/components/UpdateForm.vue";

const user = {
  firstName: "John",
  lastName: "Doe",
  email: "john.doe@example.com",
  address: "123 Main Street",
};

const fields = [
  { name: "firstName", label: "First Name", type: "text", placeholder: "Enter First Name" },
  { name: "lastName", label: "Last Name", type: "text", placeholder: "Enter Last Name" },
  { name: "email", label: "Email", type: "email", placeholder: "Enter Email" },
  { name: "address", label: "Address", type: "text", placeholder: "Enter Address" },
];

const showForm = ref(false);
const selectedData = ref<Record<string, any>>({});

const openUpdateForm = (data: Record<string, any>) => {
  selectedData.value = { ...data }; // Truyền dữ liệu người dùng vào form
  showForm.value = true;
};

const closeForm = () => {
  showForm.value = false;
};

const handleUpdate = (updatedData: Record<string, any>) => {
  console.log("Updated Data:", updatedData); // Dữ liệu sau khi cập nhật
  closeForm();
};
</script>

```
```
<script setup lang="ts">
import { reactive, watch } from "vue";
import MTextField from "@/components/mocules/MTextField/index.vue";

interface Field {
  name: string;
  label: string;
  type: string;
  placeholder?: string;
}

interface Props {
  data: Record<string, any>; // Dữ liệu được truyền vào
  fields: Field[]; // Cấu hình các trường
}

const props = defineProps<Props>();
const emits = defineEmits<{
  (e: "update", updatedData: Record<string, any>): void;
  (e: "cancel"): void;
}>();

// Sao chép dữ liệu từ props.data vào formData để quản lý
const formData = reactive({ ...props.data });

watch(
  () => props.data,
  (newData) => {
    Object.assign(formData, newData); // Đồng bộ dữ liệu khi props.data thay đổi
  }
);

const onUpdate = () => {
  emits("update", formData); // Emit dữ liệu khi nhấn update
};

const onCancel = () => {
  emits("cancel");
};
</script>

<template>
  <v-row>
    <!-- Duyệt qua danh sách fields để render input -->
    <v-col cols="12" md="6" v-for="field in fields" :key="field.name">
      <MTextField
        v-model="formData[field.name]"
        :type="field.type"
        :label="field.label"
        :placeholder="field.placeholder || ''"
      />
    </v-col>
    <v-col cols="12" md="6">
      <v-btn @click="onUpdate">Update</v-btn>
      <v-btn @click="onCancel" color="red">Cancel</v-btn>
    </v-col>
  </v-row>
</template>

```
```
<template>
  <v-container>
    <v-data-table :items="users" :headers="headers">
      <template #item.actions="{ item }">
        <v-btn @click="openUpdateForm(item.id)">Update</v-btn>
      </template>
    </v-data-table>

    <UpdateForm
      v-if="showUpdateForm"
      :id="selectedId"
      :fields="fields"
      @update="handleUpdate"
      @cancel="closeUpdateForm"
      @delete="handleDelete"
    />
  </v-container>
</template>

<script setup lang="ts">
import { ref } from "vue";
import UpdateForm from "@/components/UpdateForm.vue";

const users = ref([
  { id: "1", firstName: "John", lastName: "Doe", email: "john@example.com", address: "123 Street" },
  { id: "2", firstName: "Jane", lastName: "Smith", email: "jane@example.com", address: "456 Avenue" },
]);

const headers = [
  { text: "First Name", value: "firstName" },
  { text: "Last Name", value: "lastName" },
  { text: "Email", value: "email" },
  { text: "Actions", value: "actions", sortable: false },
];

const fields = [
  { name: "firstName", label: "First Name", type: "text", placeholder: "Enter First Name" },
  { name: "lastName", label: "Last Name", type: "text", placeholder: "Enter Last Name" },
  { name: "email", label: "Email", type: "email", placeholder: "Enter Email" },
  { name: "address", label: "Address", type: "text", placeholder: "Enter Address" },
];

const showUpdateForm = ref(false);
const selectedId = ref("");

const openUpdateForm = (id: string) => {
  selectedId.value = id;
  showUpdateForm.value = true;
};

const closeUpdateForm = () => {
  showUpdateForm.value = false;
};

const handleUpdate = (updatedData: Record<string, any>) => {
  console.log("Updated Data:", updatedData);
  closeUpdateForm();
};

const handleDelete = () => {
  console.log("Record deleted");
  closeUpdateForm();
};
</script>

```
```
<script setup lang="ts">
import { inject, ref, reactive, onMounted } from "vue";
import { Key } from "@/consts";
import Editor from "@/components/molecules/Editor/index.vue";

interface Field {
  name: string;
  label: string;
  type: string;
  placeholder?: string;
}

interface Props {
  id: string; // ID được truyền vào khi click update
  fields: Field[];
}

const $app = inject(Key);
if (!$app) throw new Error("No app provided");

const emits = defineEmits<{
  (e: "update", data: Record<string, any>): void;
  (e: "cancel"): void;
  (e: "delete"): void;
}>();

const props = defineProps<Props>();
const loading = ref(false);
const formData = reactive<Record<string, any>>({}); // Trạng thái form động

const loadData = async () => {
  try {
    loading.value = true;

    // Gọi API để lấy dữ liệu dựa trên ID
    const response = await $app.$api.get(`/api/v1/user/${props.id}`);
    Object.assign(formData, response.data); // Gán dữ liệu từ API vào formData
  } catch (error) {
    console.error("Failed to load data:", error);
  } finally {
    loading.value = false;
  }
};

const onUpdated = async () => {
  emits("update", formData); // Emit dữ liệu khi nhấn update
};

const onDeletedClicked = async () => {
  emits("delete");
};

const onCanceled = () => {
  emits("cancel");
};

onMounted(async () => {
  await loadData(); // Load dữ liệu khi component được mount
});
</script>

<template>
  <Editor :loading="loading" @update="onUpdated" @cancel="onCanceled" @delete="onDeletedClicked">
    <v-row>
      <!-- Form động dựa trên danh sách fields -->
      <v-col cols="12" md="6" v-for="field in props.fields" :key="field.name">
        <MTextField
          v-model="formData[field.name]"
          :type="field.type"
          :label="field.label"
          :placeholder="field.placeholder || ''"
        />
      </v-col>
    </v-row>
  </Editor>
</template>

```
----------------------------------updatae --------------------------------
```
<template>
  <RegistForm :fields="fields" @regist="handleRegist" />
</template>

<script setup>
import RegistForm from '~/components/RegistForm.vue';
import * as yup from 'yup';

const fields = [
  { name: "name", label: "Name", placeholder: "Enter Name", validation: yup.string().required("Name is required") },
  { name: "price", label: "Price", placeholder: "Enter Price", validation: yup.string().required("Price is required") },
  { name: "type", label: "Type", placeholder: "Enter Type", validation: yup.string().required("Type is required") },
];

const handleRegist = (data) => {
  console.log("Submitted Data:", data);
};
</script>

```
```
Nhan field tu props 
<script setup lang="ts">
import { useForm, useField } from "vee-validate";
import * as yup from "yup";
import MTextField from "~/components/mocules/MTextField/index.vue";

interface Field {
  name: string;
  label: string;
  placeholder: string;
  validation: yup.StringSchema<string>;
}

// Nhận `fields` và `registData` từ props
const props = defineProps<{
  fields: Field[];
}>();

// Xây dựng schema xác thực dựa trên `fields` từ props
const schema = yup.object(
  props.fields.reduce((acc, field) => {
    acc[field.name] = field.validation;
    return acc;
  }, {} as Record<string, yup.AnySchema>)
);

// Sử dụng Vee-Validate để quản lý form
const { handleSubmit, errors } = useForm({
  validationSchema: schema,
  initialValues: props.fields.reduce((acc, field) => {
    acc[field.name] = ""; // Giá trị mặc định là chuỗi rỗng
    return acc;
  }, {} as Record<string, string>),
});

// Xử lý sự kiện khi form được gửi
const emits = defineEmits<{
  (e: "regist", data: Record<string, string>): void;
}>();
const onSubmit = handleSubmit(values => {
  console.log("Form Values:", values);
  emits("regist", values);
});

// Tạo trạng thái động cho từng trường từ props.fields
const fieldsState = props.fields.map(field => ({
  ...field,
  field: useField(field.name),
}));
</script>

<template>
  <v-row>
    <!-- Duyệt qua các trường và hiển thị động -->
    <v-col cols="12" md="6" v-for="fieldState in fieldsState" :key="fieldState.name">
      <MTextField
        v-model="fieldState.field.value"
        :label="fieldState.label"
        :placeholder="fieldState.placeholder"
        :error-message="fieldState.field.errorMessage"
        @blur="fieldState.field.handleBlur"
      />
    </v-col>

    <v-col cols="12" md="6">
      <v-btn @click="onSubmit">Regist</v-btn>
    </v-col>
  </v-row>
</template>

```
```
<script setup lang="ts">
import regist from '~/components/search/index.vue';
import * as yup from 'yup';

// Sử dụng Record<string, string> để phù hợp với form động
 type RegistData = Record<string, string>;

 class Controller {
    public registData: RegistData;
    constructor() {
        // Khởi tạo các giá trị ban đầu dưới dạng chuỗi rỗng
        this.registData = {
            name: '',
            price: '',
            type: '',
        };
    }
}
 const useController = () => new Controller();
const { registData } = useController();

// Mảng fields định nghĩa các trường động
const fields = [
    { name: 'name', label: 'Name', placeholder: 'Enter Name', validation: yup.string().required("Phai nhap") },
    { name: 'price', label: 'Price', placeholder: 'Enter Price', validation: yup.string().required("Can nhap") },
    { name: 'type', label: 'Type', placeholder: 'Enter Type', validation: yup.string().required("Sao khong nhap") },
];
</script>

<template>
    <!-- Truyền fields và registData vào component con -->
    <regist :fields="fields" :regist-data="registData" />
</template>

```
```
useController
export interface Props {
    label? : string;
    placeholder? : string;
    errorMessage? : string;
    hideDetail? : boolean;
    autoFocus? : boolean;
    maxlength? : number;
}

export interface MTextFieldInterface { 
    focus: () => void;
    isError: () => boolean;
}

class Controller {
    private props;
    public templateRef: Ref<unknown>;

    constructor( props: Props) {
        this.props = props;
        this.templateRef = ref(null);
    }

    public focus = (): void => {
        (this.templateRef.value as MTextFieldInterface).focus();
    };

    public isError = (): boolean => {
        if(this.props.errorMessage) {
            return true;
        }
        return false;
    };
}

export const useController = ( props: Props) => new Controller( props); 
```
```
<script setup lang="ts">
    import { inject } from 'vue';
    import { type Props, useController as ctrl, type MTextFieldInterface } from './useController';


    const props = withDefaults(defineProps<Props>(), {
        label: '',
        placeholder: '',
        errorMessage: '',
        hideDetail: false,
        autoFocus: false,
        maxlength: undefined
    });

    const { templateRef, focus, isError } = ctrl( props);

    const model = defineModel<unknown>({ default: () => undefined});

    defineExpose<MTextFieldInterface>({
        focus: focus,
        isError: isError,
    });
</script>
<template>
    <v-row dense>
        <template v-if="props.label">
            <v-col cols="12" md="2" class=" d-flex justify-start" :class="{ 'mt-n5' :hideDetail, 'align-center' :hideDetail }">
                <v-label>{{ props.label }}</v-label>
            </v-col>
        </template>
        <v-col cols="12" md="10" class=" d-flex justify-start" :class="{ 'mt-n5' :hideDetail, 'align-center' :hideDetail }">
            <v-text-field
                :ref="(el:any) => (templateRef = el)"
                v-model="model"
                :label="props.label"
                :placeholder="props.placeholder"
                :error-messages="props.errorMessage"
                :hide-detail="props.hideDetail"
                :autofocus="props.autoFocus"
                :maxlength="props.maxlength"
            />
        </v-col>
    </v-row>
</template>
```
```
<script setup lang="ts">
import * as yup from "yup";
import MTextField from "~/components/mocules/MTextField/index.vue";
import { ref, reactive } from "vue";

type RegistData = Record<string, string>;
interface Field {
    name: string;
    label: string;
    placeholder: string;
    validation: yup.StringSchema<string>;
}

const emits = defineEmits<{
    (e: 'regist'): void;
}>();

interface Props {
    fields: Field[];
    registData: RegistData;
}

class RegistController {
    public fields: Field[];
    public props;
    public registData: Ref<RegistData>;
    public errors = reactive<Record<string, string>>({});

    constructor(props: Props) {
        this.fields = props.fields;
        this.props = props;
        this.registData = ref<RegistData>({ ...this.props.registData });
        this.resetErrors();

        // Ràng buộc các phương thức
        this.validateAll = this.validateAll.bind(this);
        this.regist = this.regist.bind(this);
    }

    async validateField(name: string) {
        const field = this.fields.find(f => f.name === name);
        if (!field) return;
        try {
            await field.validation.validate(this.registData.value[name]);
            this.errors[name] = "";
        } catch (error: any) {
            this.errors[name] = error.message;
        }
    }

    async validateAll() {
        await Promise.all(this.fields.map(field => this.validateField(field.name)));
        return Object.values(this.errors).every(error => !error);
    }

    resetErrors() {
        this.fields.forEach(field => {
            this.errors[field.name] = "";
        });
    }

    async regist() {
        if (await this.validateAll()) {
            console.log("Data:", this.registData.value);
            emits('regist');
        } else {
            console.warn("Validation failed");
        }
    }
}

const props = defineProps<Props>();
const controller = new RegistController(props);

const { fields, registData, errors, validateField } = controller;
const regist = controller.regist;
</script>
<template>
    <v-row>
        <v-col cols="12" md="6" v-for="field in fields" :key="field.name">
            <MTextField
                v-model="registData[field.name]"
                :label="field.label"
                :placeholder="field.placeholder"
                :error-message="errors[field.name]"
                @blur="validateField(field.name)"
            />
        </v-col>
        <v-col cols="12" md="6">
            <v-btn @click="regist">Regist</v-btn>
        </v-col>
    </v-row>
</template>

```
