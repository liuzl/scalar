<script lang="ts" setup>
import { CodeMirror } from '@scalar/use-codemirror'

import {
  getExampleFromSchema,
  mapFromObject,
  prettyPrintJson,
} from '../../../../helpers'
import SelectExample from './SelectExample.vue'

defineProps<{
  response:
    | undefined
    | {
        examples?: Record<string, any>
        example?: any
        schema?: any
      }
}>()

const rules = ['oneOf', 'anyOf', 'not']

const mergeAllObjects = (items: Record<any, any>[]): any => {
  return items.reduce((acc, object) => {
    return {
      ...acc,
      ...object,
    }
  }, {})
}
</script>

<template>
  <!-- Multiple examples -->
  <template
    v-if="response?.examples && Object.keys(response?.examples).length > 1">
    <SelectExample :examples="response?.examples" />
  </template>
  <!-- An array, but just one example -->
  <template
    v-else-if="
      response?.examples && Object.keys(response?.examples).length === 1
    ">
    <CodeMirror
      :content="
        prettyPrintJson(mapFromObject(response?.examples)[0].response.response)
      "
      :languages="['json']"
      readOnly />
  </template>
  <!-- Single example -->
  <template v-else>
    <div v-if="response?.example">
      <CodeMirror
        :content="prettyPrintJson(response?.example)"
        :languages="['json']"
        readOnly />
    </div>
    <div v-else-if="response?.schema">
      <!-- Single Schema -->
      <CodeMirror
        v-if="response?.schema.type"
        :content="
          prettyPrintJson(
            getExampleFromSchema(
              response?.schema,

              {
                emptyString: '…',
                mode: 'read',
              },
            ),
          )
        "
        :languages="['json']"
        readOnly />
      <!-- oneOf, anyOf, not … -->
      <template
        v-for="rule in rules"
        :key="rule">
        <div
          v-if="
            response?.schema[rule] &&
            (response?.schema[rule].length > 1 || rule === 'not')
          "
          class="rule">
          <div class="rule-title">
            {{ rule }}
          </div>
          <ol class="rule-items">
            <li
              v-for="(example, index) in response?.schema[rule]"
              :key="index"
              class="rule-item">
              <CodeMirror
                :content="
                  prettyPrintJson(
                    getExampleFromSchema(example, {
                      emptyString: '…',
                      mode: 'read',
                    }),
                  )
                "
                :languages="['json']"
                readOnly />
            </li>
          </ol>
        </div>
        <CodeMirror
          v-else-if="
            response?.schema[rule] && response?.schema[rule].length === 1
          "
          :content="
            prettyPrintJson(
              getExampleFromSchema(response?.schema[rule][0], {
                emptyString: '…',
                mode: 'read',
              }),
            )
          "
          :languages="['json']"
          readOnly />
      </template>
      <!-- allOf-->
      <CodeMirror
        v-if="response?.schema['allOf']"
        :content="
          prettyPrintJson(
            mergeAllObjects(
              response?.schema['allOf'].map((schema: any) =>
                getExampleFromSchema(schema, {
                  emptyString: '…',
                  mode: 'read',
                }),
              ),
            ),
          )
        "
        :languages="['json']"
        readOnly />
    </div>
    <div
      v-if="!response?.example && !response?.schema"
      class="empty-state">
      No Body
    </div>
  </template>
</template>

<style scoped>
.empty-state {
  margin: 10px 0 10px 12px;
  text-align: center;
  font-size: var(--theme-micro, var(--default-theme-micro));
  min-height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--theme-radius-lg, var(--default-theme-radius-lg));
  color: var(--theme-color-2, var(--default-theme-color-2));
}

.rule-title {
  font-family: var(--theme-font-code, var(--default-theme-font-code));
  color: var(--theme-color-1, var(--default-theme-color-1));
  display: inline-block;
  margin: 12px 0 6px;
  border-radius: var(--theme-radius, var(--default-theme-radius));
}

.rule {
  margin: 0 12px 0;
  border-radius: var(--theme-radius-lg, var(--default-theme-radius-lg));
}

.rule-items {
  counter-reset: list-number;
  display: flex;
  flex-direction: column;
  gap: 12px;
  border-left: 1px solid
    var(--theme-border-color, var(--default-theme-border-color));
  padding: 12px 0 12px;
}
.rule-item {
  counter-increment: list-number;
  border: 1px solid var(--theme-border-color, var(--default-theme-border-color));
  border-radius: var(--theme-radius-lg, var(--default-theme-radius-lg));
  overflow: hidden;
  margin-left: 24px;
}
.rule-item:before {
  /* content: counter(list-number); */
  border: 1px solid var(--theme-border-color, var(--default-theme-border-color));
  border-top: 0;
  border-right: 0;
  content: ' ';
  display: block;
  width: 24px;
  height: 6px;
  border-radius: 0 0 0 var(--theme-radius-lg, var(--default-theme-radius-lg));
  margin-top: 6px;
  color: var(--theme-color-2, var(--default-theme-color-2));
  transform: translateX(-25px);
  color: var(--theme-color-1, var(--default-theme-color-1));
  position: absolute;
}
</style>
