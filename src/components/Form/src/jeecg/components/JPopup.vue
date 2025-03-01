<!--popup组件-->
<template>
    <div class="components-input-demo-presuffix" v-if="avalid">
        <!--输入框-->
        <a-input @click="handleOpen" v-model:value="showText" :placeholder="placeholder" readOnly v-bind="attrs">
            <template #prefix>
                <Icon icon="ant-design:cluster-outlined"></Icon>
            </template>
            <template #suffix>
                <Icon icon="ant-design:close-circle-outlined" @click="handleEmpty" title="清空" v-if="showText"></Icon>
            </template>
        </a-input>
        <!--popup弹窗-->
        <JPopupOnlReportModal @register="regModal" :code="code" :multi="multi" :sorter="sorter" :groupId="uniqGroupId" :param="param" @ok="callBack"></JPopupOnlReportModal>
    </div>
</template>
<script lang="ts">
    import JPopupOnlReportModal from './modal/JPopupOnlReportModal.vue'
    import {defineComponent, ref, reactive, onMounted, watchEffect, watch, computed, unref} from 'vue';
    import {useModal} from '/@/components/Modal';
    import {propTypes} from "/@/utils/propTypes";
    import {useAttrs} from "/@/hooks/core/useAttrs";
    import {useMessage} from '/@/hooks/web/useMessage';

    export default defineComponent({
        name: 'JPopup',
        components: {
          JPopupOnlReportModal
        },
        inheritAttrs: false,
        props: {
            code: propTypes.string.def(''),
            value: propTypes.string.def(''),
            sorter: propTypes.string.def(''),
            width: propTypes.number.def(1200),
            placeholder: propTypes.string.def('请选择'),
            multi: propTypes.bool.def(false),
            param: propTypes.object.def({}),
            spliter: propTypes.string.def(','),
            groupId: propTypes.string.def(''),
            formElRef: propTypes.object,
            setFieldsValue: propTypes.func,
            fieldConfig: {
                type: Array,
                default: () => []
            },
        },
        emits: ['update:value','register'],
        setup(props, {emit, refs}) {
            const {createMessage} = useMessage();
            const attrs = useAttrs();
            //pop是否展示
            const avalid = ref(true);
            const showText = ref('');
            //注册model
            const [regModal, {openModal}] = useModal();
            //表单值
            let {groupId, code, fieldConfig} = props;
            //唯一分组groupId
            const uniqGroupId = computed(() => groupId ? `${groupId}_${code}_${fieldConfig[0]['source']}_${fieldConfig[0]['target']}` : '');
            /**
             * 判断popup配置项是否正确
             */
            onMounted(() => {
                if (props.fieldConfig.length == 0) {
                    createMessage.error('popup参数未正确配置!');
                    avalid.value = false;
                }
            });
            /**
             * 监听value数值
             */
            watch(
                () => props.value,
                (val) => {
                    showText.value = val && val.length>0 ? val.split(props.spliter).join(',') : '';
                },
                {immediate: true}
            );

            /**
             * 打开pop弹出框
             */
            function handleOpen() {
                !props.disabled && openModal(true);
            }

            /**
             * TODO 清空
             */
            function handleEmpty() {
                showText.value = '';
            }

            /**
             * 传值回调
             */
            function callBack(rows) {
                let {fieldConfig} = props;
                //匹配popup设置的回调值
                let values = {};
                for (let item of fieldConfig) {
                    let val = rows.map(row => row[item.source]).join(',');
                    item.target.split(",").forEach(target => {
                        values[target] = val;
                    });
                }
                //传入表单示例方式赋值
                props.formElRef && props.formElRef.setFieldsValue(values);
                //传入赋值方法方式赋值
                props.setFieldsValue && props.setFieldsValue(values);
            }

            return {
                showText,
                avalid,
                uniqGroupId,
                attrs,
                regModal,
                handleOpen,
                handleEmpty,
                callBack,
            };
        },
    });
</script>
<style lang="less" scoped>
    .components-input-demo-presuffix .anticon-close-circle {
        cursor: pointer;
        color: #ccc;
        transition: color 0.3s;
        font-size: 12px;
    }

    .components-input-demo-presuffix .anticon-close-circle:hover {
        color: #f5222d;
    }

    .components-input-demo-presuffix .anticon-close-circle:active {
        color: #666;
    }
</style>
