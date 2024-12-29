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
