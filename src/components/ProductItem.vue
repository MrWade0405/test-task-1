<template>
  <tr>
    <td>{{ this.item.artnumber }}</td>
    <td>{{ this.item.name }}</td>
    <td>{{ this.item.brand }}</td>
    <td>{{ this.item.weight }} г</td>
    <td>{{ this.item.quantity }} г</td>
    <td>{{ this.item.price }} грн</td>
    <td>{{stock}}</td>
  </tr>
</template>

<script>

export default {
  name: 'productItem',
  props: ['item'],
  computed: {
    stock() {
      if (this.item.stock == 1) return 'Есть в наличии';
      else return 'Нет в наличии';
    }
  }
}
</script>
