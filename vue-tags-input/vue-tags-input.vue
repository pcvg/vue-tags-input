<!--
  Entry Point for the component.
  The file contains the template and includes the script and style files.
-->

<template>
  <div
    class="vue-tags-input"
    :class="[{ 'ti-disabled': disabled }, { 'ti-focus': focused }, $attrs.class]"
    :style="$attrs.style"
  >
    <div class="ti-input">
      <draggable
        :is="isDraggable ? 'draggable' : 'ul'"
        tag="ul"
        group="tags"
        class="ti-tags"
        :list="tagsCopy"
        item-key="text"
        :handle="draggableHandle ? '.tag-handle' : ''"
        ghost-class="ghost-tag"
        drag-class="drag-tag"
        @start="drag=true"
        @end="drag=false; tagOrderChanged()"
      >
        <template #item="{ element, index }">
          <li
            :key="index"
            :style="element.style"
            :class="[
              { 'ti-editing': tagsEditStatus[index] },
              element.tiClasses,
              element.classes,
              { 'ti-deletion-mark': isMarked(index) }
            ]"
            tabindex="0"
            class="ti-tag item"
            @click="$emit('tag-clicked', { element, index })">
            <div class="ti-content">
              <slot
                v-if="$slots['tag-handle'] && draggableHandle"
                name="tag-handle"
                class="tag-handle"/>
              <span
                v-if="!$slots['tag-handle'] && draggableHandle"
                class="tag-handle">::</span>
              <div
                v-if="$slots['tag-left']"
                class="ti-tag-left"
              >
                <slot
                  name="tag-left"
                  :tag="element"
                  :index="index"
                  :edit="tagsEditStatus[index]"
                  :perform-save-edit="performSaveTag"
                  :perform-delete="performDeleteTag"
                  :perform-cancel-edit="cancelEdit"
                  :perform-open-edit="performEditTag"
                  :deletion-mark="isMarked(index)"
                />
              </div>
              <div :ref="setTagCenter" class="ti-tag-center">
                <span
                  v-if="!$slots['tag-center']"
                  :class="{ 'ti-hidden': tagsEditStatus[index] }"
                  @click="performEditTag(index)">
                  {{ element.text }}
                </span>
                <tag-input
                  v-if="!$slots['tag-center']"
                  :scope="{
                    edit: tagsEditStatus[index],
                    maxlength,
                    element,
                    index,
                    validateTag: createChangedTag,
                    performCancelEdit: cancelEdit,
                    performSaveEdit: performSaveTag,
                  }"
                />
                <slot
                  name="tag-center"
                  :tag="element"
                  :index="index"
                  :maxlength="maxlength"
                  :edit="tagsEditStatus[index]"
                  :perform-save-edit="performSaveTag"
                  :perform-delete="performDeleteTag"
                  :perform-cancel-edit="cancelEdit"
                  :validate-tag="createChangedTag"
                  :perform-open-edit="performEditTag"
                  :deletion-mark="isMarked(index)"
                />
              </div>
              <div
                v-if="$slots['tag-right']"
                class="ti-tag-right"
              >
                <slot
                  name="tag-right"
                  :tag="element"
                  :index="index"
                  :edit="tagsEditStatus[index]"
                  :perform-save-edit="performSaveTag"
                  :perform-delete="performDeleteTag"
                  :perform-cancel-edit="cancelEdit"
                  :perform-open-edit="performEditTag"
                  :deletion-mark="isMarked(index)"
                />
              </div>
            </div>
            <div class="ti-actions">
              <!-- dont use v-if and v-else here -> different event calling on click?! -->
              <i
                v-if="!$slots['tag-actions']"
                v-show="tagsEditStatus[index]"
                class="ti-icon-undo"
                @click="cancelEdit(index)"
              />
              <i
                v-if="!$slots['tag-actions']"
                v-show="!tagsEditStatus[index]"
                class="ti-icon-close"
                @click="performDeleteTag(index)"
              />
              <slot
                v-if="$slots['tag-actions']"
                name="tag-actions"
                :tag="element"
                :index="index"
                :edit="tagsEditStatus[index]"
                :perform-save-edit="performSaveTag"
                :perform-delete="performDeleteTag"
                :perform-cancel-edit="cancelEdit"
                :perform-open-edit="performEditTag"
                :deletion-mark="isMarked(index)"
              />
            </div>
          </li>
        </template>
      </draggable>
      <div class="ti-new-tag-input-wrapper">
        <input
          ref="newTagInput"
          v-bind="$attrs"
          :class="[createClasses(newTag, tags, validation, isDuplicate)]"
          :placeholder="placeholder"
          :value="newTag"
          :maxlength="maxlength"
          :disabled="disabled"
          :type="inputType"
          size="1"
          class="ti-new-tag-input"
          @keydown="performAddTags(
            filteredAutocompleteItems[selectedItem] || newTag, $event
          )"
          @paste="addTagsFromPaste"
          @keydown.delete="invokeDelete"
          @keydown.tab="performBlur"
          @keydown.up="selectItem($event, 'before')"
          @keydown.down="selectItem($event, 'after')"
          @input="updateNewTag"
          @focus="focused = true"
          @click="performClick($event)"
        >
      </div>
    </div>
    <slot name="between-elements" />
    <div
      v-if="autocompleteOpen"
      class="ti-autocomplete"
      @mouseout="selectedItem = null"
    >
      <slot name="autocomplete-header" />
      <ul>
        <li
          v-for="(item, index) in filteredAutocompleteItems"
          :key="index"
          :style="item.style"
          :class="[
            item.tiClasses,
            item.classes,
            { 'ti-selected-item': isSelected(index) }
          ]"
          class="ti-item"
          @mouseover="disabled ? false : selectedItem = index"
        >
          <div
            v-if="!$slots['autocomplete-item']"
            @click="performAddTags(item, undefined, 'autocomplete')"
          >
            {{ item.text }}
          </div>
          <slot
            v-else
            name="autocomplete-item"
            :item="item"
            :index="index"
            :perform-add="item => performAddTags(item, undefined, 'autocomplete')"
            :selected="isSelected(index)"
          />
        </li>
      </ul>
      <slot name="autocomplete-footer" />
    </div>
  </div>
</template>

<!-- js and scss resources → I separated it into different files, because they became huge -->
<script src="./vue-tags-input.js"></script>
<style lang="scss" src="./vue-tags-input.scss" scoped></style>
