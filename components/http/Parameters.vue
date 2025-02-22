<template>
  <AppSection label="parameters">
    <div
      class="
        bg-primary
        border-b border-dividerLight
        flex flex-1
        top-upperSecondaryStickyFold
        pl-4
        z-10
        sticky
        items-center
        justify-between
      "
    >
      <label class="font-semibold text-secondaryLight">
        {{ $t("request.parameter_list") }}
      </label>
      <div class="flex">
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          to="https://docs.hoppscotch.io/features/parameters"
          blank
          :title="$t('app.wiki')"
          svg="help-circle"
        />
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('action.clear_all')"
          svg="trash-2"
          @click.native="clearContent"
        />
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('add.new')"
          svg="plus"
          @click.native="addParam"
        />
      </div>
    </div>
    <div
      v-for="(param, index) in params$"
      :key="`param-${index}`"
      class="divide-x divide-dividerLight border-b border-dividerLight flex"
    >
      <SmartEnvInput
        v-if="EXPERIMENTAL_URL_BAR_ENABLED"
        v-model="param.key"
        :placeholder="$t('count.parameter', { count: index + 1 })"
        styles="
          bg-transparent
          flex
          flex-1
          py-1
          px-4
        "
        @change="
          updateParam(index, {
            key: $event,
            value: param.value,
            active: param.active,
          })
        "
      />
      <input
        v-else
        class="bg-transparent flex flex-1 py-2 px-4"
        :placeholder="$t('count.parameter', { count: index + 1 })"
        :name="'param' + index"
        :value="param.key"
        autofocus
        @change="
          updateParam(index, {
            key: $event.target.value,
            value: param.value,
            active: param.active,
          })
        "
      />
      <SmartEnvInput
        v-if="EXPERIMENTAL_URL_BAR_ENABLED"
        v-model="param.value"
        :placeholder="$t('count.value', { count: index + 1 })"
        styles="
          bg-transparent
          flex
          flex-1
          py-1
          px-4
        "
        @change="
          updateParam(index, {
            key: param.key,
            value: $event,
            active: param.active,
          })
        "
      />
      <input
        v-else
        class="bg-transparent flex flex-1 py-2 px-4"
        :placeholder="$t('count.value', { count: index + 1 })"
        :name="'value' + index"
        :value="param.value"
        @change="
          updateParam(index, {
            key: param.key,
            value: $event.target.value,
            active: param.active,
          })
        "
      />
      <span>
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="
            param.hasOwnProperty('active')
              ? param.active
                ? $t('action.turn_off')
                : $t('action.turn_on')
              : $t('action.turn_off')
          "
          :svg="
            param.hasOwnProperty('active')
              ? param.active
                ? 'check-circle'
                : 'circle'
              : 'check-circle'
          "
          color="green"
          @click.native="
            updateParam(index, {
              key: param.key,
              value: param.value,
              active: param.hasOwnProperty('active') ? !param.active : false,
            })
          "
        />
      </span>
      <span>
        <ButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="$t('action.remove')"
          svg="trash"
          color="red"
          @click.native="deleteParam(index)"
        />
      </span>
    </div>
    <div
      v-if="params$.length === 0"
      class="flex flex-col text-secondaryLight p-4 items-center justify-center"
    >
      <span class="text-center pb-4">
        {{ $t("empty.parameters") }}
      </span>
      <ButtonSecondary
        :label="$t('add.new')"
        svg="plus"
        filled
        @click.native="addParam"
      />
    </div>
  </AppSection>
</template>

<script lang="ts">
import { defineComponent } from "@nuxtjs/composition-api"
import { HoppRESTParam } from "~/helpers/types/HoppRESTRequest"
import { useReadonlyStream } from "~/helpers/utils/composables"
import {
  restParams$,
  addRESTParam,
  updateRESTParam,
  deleteRESTParam,
  deleteAllRESTParams,
} from "~/newstore/RESTSession"
import { useSetting } from "~/newstore/settings"

export default defineComponent({
  setup() {
    return {
      params$: useReadonlyStream(restParams$, []),
      EXPERIMENTAL_URL_BAR_ENABLED: useSetting("EXPERIMENTAL_URL_BAR_ENABLED"),
    }
  },
  watch: {
    params$: {
      handler(newValue) {
        if (
          (newValue[newValue.length - 1]?.key !== "" ||
            newValue[newValue.length - 1]?.value !== "") &&
          newValue.length
        )
          this.addParam()
      },
      deep: true,
    },
  },
  // mounted() {
  //   if (!this.params$?.length) {
  //     this.addParam()
  //   }
  // },
  methods: {
    addParam() {
      addRESTParam({ key: "", value: "", active: true })
    },
    updateParam(index: number, item: HoppRESTParam) {
      updateRESTParam(index, item)
    },
    deleteParam(index: number) {
      deleteRESTParam(index)
    },
    clearContent() {
      deleteAllRESTParams()
    },
  },
})
</script>
