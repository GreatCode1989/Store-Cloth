<template>
<div
    :class="{ 'btn-primary': !isPartIdAdded, 'btn-warning': isPartIdAdded}"
    class="offer-heart ms-1"
    :style="{ color: isPartIdAdded ? 'green' : 'white' }" 
  >
    <i @click="handleButtonClick" class="fas fa-heart heart-icon"></i>
  </div>
</template>


<script setup>
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import { ref, onMounted } from "vue";

const router = useRouter();
const store = useStore();

const props = defineProps({
  size: {
    type: Boolean,
    default: false,
  },
  item: {
    type: Object,
    required: true,
  },
});

const storedToken = localStorage.getItem("tokenData");
const tokenData = storedToken ? JSON.parse(storedToken) : null;

const isPartIdAdded = ref(true);

onMounted(() => {
  if (tokenData && props.item && props.item._id) {
    const partId = props.item._id;

    store
      .dispatch("checkPartId", {
        partId,
        userId: tokenData.id,
      })
      .then((result) => {
        if (typeof result === "boolean") {
          isPartIdAdded.value = result;
        } else if (
          result &&
          typeof result === "object" &&
          "isPartIdAdded" in result
        ) {
          isPartIdAdded.value = result.isPartIdAdded;
        } else {
          console.error("Invalid response format:", result);
        }
      });
  }
});

const handleButtonClick = () => {
  if (isPartIdAdded.value) {
    router.push({ name: "cart" });
  } else {
    addToCart();
  }
};

const addToCart = () => {
  if (tokenData && props.item && props.item._id) {
    store
      .dispatch("addToCart", {
        partId: props.item._id,
        username: tokenData.username,
        userId: tokenData.id,
      })
      .then((result) => {
        if (result) {
          isPartIdAdded.value = true;
          const cart = result.data;
          console.log("Корзина успешно обновлена:", cart);
          router.push({ name: "cart" });
          return cart;
        } else {
          console.error("Произошла ошибка при добавлении в корзину.");
        }
      });
  } else {
    console.error(
      "Данные пользователя отсутствуют или отсутствует свойство id у item. Невозможно добавить в корзину."
    );
  }
};
</script>

<style scoped>

.heart-icon {
  font-size: 25px; 
  border-radius: 50%; 
  padding-top: 9px;
  cursor: pointer;
}

.heart-icon :hover {
  background: white;
}
</style>
