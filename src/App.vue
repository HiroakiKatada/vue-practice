<template>
  <div class="p-10">
    <!-- 縦・横・高さの入力フィールド -->
    <div class="flex gap-4 mb-8">
      <div class="grid gap-2">
        <label class="font-semibold">縦</label>
        <div class="flex items-end gap-2">
          <input type="number" v-model="length" step="10" class="w-32 border rounded p-2" />
          <span>mm</span>
        </div>
      </div>
      <div class="grid gap-2">
        <label class="font-semibold">横</label>
        <div class="flex items-end gap-2">
          <input type="number" v-model="width" step="10" class="w-32 border rounded p-2" />
          <span>mm</span>
        </div>
      </div>
      <div class="grid gap-2">
        <label class="font-semibold">高さ</label>
        <div class="flex items-end gap-2">
          <input type="number" v-model="height" step="10" class="w-32 border rounded p-2" />
          <span>mm</span>
        </div>
      </div>
    </div>

    <!-- 重さの入力フィールド -->
    <div class="grid gap-2 mb-8">
      <label class="font-semibold">重さ</label>
      <div class="flex items-end gap-2">
        <input type="number" v-model="weight" step="100" class="w-32 border rounded p-2" />
        <span>g</span>
      </div>
    </div>

    <!-- 発送サイズの表示 -->
    <div class="grid gap-2 mb-8">
      <label class="font-semibold">発送サイズ</label>
      <div class="text-zinc-700 bg-zinc-100 px-4 py-2 rounded w-fit">
        {{ result.size }}
      </div>
    </div>

    <!-- 発送方法の選択 -->
    <div class="grid gap-2 mb-8">
      <div class="font-semibold">発送方法</div>
      <div class="flex gap-4">
        <label class="flex gap-2">
          <input type="radio" name="deliv" value="常温" v-model="shippingType" />
          <span>常温</span>
        </label>
        <label class="flex gap-2">
          <input type="radio" name="deliv" value="クール便" v-model="shippingType" />
          <span>冷蔵・冷凍</span>
        </label>
      </div>
    </div>

    <!-- N2発送サイズコードの表示 -->
    <div class="grid gap-2">
      <div class="font-semibold">N2発送サイズコード</div>
      <div
        class="px-4 py-2 rounded w-fit cursor-pointer bg-zinc-100 text-zinc-700"
        :class="{ 'bg-green-200': copied }"
        @click="copyToClipboard(result.number)"
      >
        {{ result.number }}
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const length = ref(200);
    const width = ref(100);
    const height = ref(400);
    const weight = ref(2000); // g単位で入力
    const shippingType = ref('常温'); // デフォルトは常温
    const copied = ref(false); // コピー状態のフラグ

    // サイズ表 (mm, g単位)
    const sizeTable = [
      { name: '60サイズ', dimensions: 600, weight: 2, number: '101' },
      { name: '80サイズ', dimensions: 800, weight: 5, number: '102' },
      { name: '100サイズ', dimensions: 1000, weight: 10, number: '103' },
      { name: '120サイズ', dimensions: 1200, weight: 15, number: '104' },
      { name: '140サイズ', dimensions: 1400, weight: 20, number: '105' },
      { name: '160サイズ', dimensions: 1600, weight: 25, number: '106' },
      { name: '180サイズ', dimensions: 1800, weight: 30, number: '107' },
      { name: '200サイズ', dimensions: 2000, weight: 30, number: '108' },
    ];

    const totalDimensions = computed(() => length.value + width.value + height.value);

    const result = computed(() => {
      let selectedSize = null;

      for (const size of sizeTable) {
        if (totalDimensions.value <= size.dimensions && weight.value <= size.weight * 1000) {
          selectedSize = size;
          break;
        }
      }

      if (selectedSize) {
        const canUseCool = shippingType.value === '常温' || selectedSize.dimensions <= 1200;
        return {
          size: selectedSize.name,
          number: canUseCool ? selectedSize.number : '-',
        };
      } else {
        return {
          size: '利用不可',
          number: '-',
        };
      }
    });

    async function copyToClipboard(text) {
      if (text !== '-') {
        try {
          await navigator.clipboard.writeText(text);
          copied.value = true;
          setTimeout(() => (copied.value = false), 500); // 色変更のリセット
        } catch (error) {
          console.error('コピーに失敗しました', error);
        }
      } else {
        console.error('コピーできる番号がありません。');
      }
    }

    return {
      length,
      width,
      height,
      weight,
      shippingType,
      result,
      sizeTable,
      copyToClipboard,
      copied,
    };
  },
};
</script>

<style scoped>
.cursor-pointer {
  cursor: pointer;
}
.w-fit {
  width: fit-content;
}
.bg-green-200 {
  background-color: #a7f3d0;
}
</style>
