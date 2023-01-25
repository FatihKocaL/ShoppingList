<template>
  <v-container class="d-flex justify-center">
    <v-card width="70vw">
      <v-card-title> Alışveriş Listesi </v-card-title>
      <v-divider></v-divider>
      <v-card-text>
        <v-row class="d-flex align-center">
          <v-col>
            <v-text-field
              v-model="requestModelProduct.name"
              hide-details
              variant="outlined"
              density="compact"
              label="Alınacak"
            >
            </v-text-field>
          </v-col>
          <v-col cols="3">
            <v-text-field
              @keyup.enter="createProduct()"
              v-model="requestModelProduct.price"
              hide-details
              variant="outlined"
              density="compact"
              label="Fiyatı"
            >
            </v-text-field>
          </v-col>
          <v-col cols="auto">
            <v-btn @click="createProduct()" color="success">Ekle</v-btn>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-table height="50vh" fixed-header>
              <thead>
                <tr>
                  <th class="text-left">Sıra</th>
                  <th class="text-left">Ürün Adı</th>
                  <th class="text-left">Ürün Fiyatı</th>
                  <th class="text-left">Satın Alındı mı ?</th>
                  <th class="text-left">İşlem</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in shoppingList" :key="item.id">
                  <td>{{ index }}</td>
                  <td>{{ item.name }}</td>
                  <td>{{ item.price }}</td>
                  <td>
                    <v-checkbox
                      @update:model-value="updateProduct(item)"
                      hide-details
                      class="pa-0"
                      v-model="item.isBuy"
                    ></v-checkbox>
                  </td>
                  <td>
                    <v-btn color="red" @click.stop="deleteProduct(item)"
                      >delete</v-btn
                    >
                  </td>
                </tr>
              </tbody>
            </v-table>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-text-field
              v-model="calculateModel.buyingTotalPrice"
              hide-details
              variant="outlined"
              density="compact"
              label="Alınanların Fiyatı"
              readonly
            >
            </v-text-field>
          </v-col>
          <v-col>
            <v-text-field
              v-model="calculateModel.notBuyingTotalPrice"
              hide-details
              variant="outlined"
              density="compact"
              label="Henüz Alınmayanların Fiyatı"
              readonly
            >
            </v-text-field>
          </v-col>
          <v-col>
            <v-text-field
              v-model="calculateModel.totalPrice"
              hide-details
              variant="outlined"
              density="compact"
              label="Toplam Fiyat"
              readonly
            >
            </v-text-field>
          </v-col>
        </v-row>
        <p style="color: red" class="text-right">
          {{ notBuyingLength }}: Adet alınacak ürün vardır.
        </p>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script lang="ts" >
import { Shopping } from "@/models/Shopping";
import { defineComponent } from "@vue/runtime-core";
import axios from "axios";
export default defineComponent({
  name: "ShoppingComponent",
  data: () => ({
    shoppingList: new Array<Shopping>(),
    requestModelProduct: new Shopping(),
    calculateModel: {
      totalPrice: 0,
      buyingTotalPrice: 0,
      notBuyingTotalPrice: 0,
    },
  }),
  methods: {
    getShoppingList() {
      axios
        .get("https://63cc06339b72d2a88e068e16.mockapi.io/api/shopping")
        .then((res: any) => {
          this.shoppingList = res.data;
          this.calculateTotalPrice();
        });
    },
    createProduct() {
      if (this.requestModelProduct.name.trim() == "") {
        alert("alan boş bırakılamaz");
      } else {
        axios
          .post(
            "https://63cc06339b72d2a88e068e16.mockapi.io/api/shopping",
            this.requestModelProduct
          )
          .then((res: any) => {
            this.getShoppingList();
            //2.yöntem (direk arrayin içine pushladık)
            // this.shoppingList.push(res.data);
          });
      }
      this.requestModelProduct = new Shopping();
    },
    deleteProduct(parametrItem: Shopping) {
      axios
        .delete(
          `https://63cc06339b72d2a88e068e16.mockapi.io/api/shopping/${parametrItem.id}`
        )
        .then((res: any) => {
          //   this.getShoppingList();
          // ikinci yöntem
          const index = this.shoppingList.findIndex(
            (x) => x.id == parametrItem.id
          );
          this.shoppingList.splice(index, 1);
        });
    },
    updateProduct(parametrItem: Shopping) {
      // short if kullanımı:
      // koşul ? doğruysa dön : yanlışsa dön
      parametrItem.isBuy = parametrItem.isBuy == true ? false : true;
      // parametrItemim isbuyı eşit ise true ? false döndür : değilse true döndür.
      axios
        .put(
          `https://63cc06339b72d2a88e068e16.mockapi.io/api/shopping/${parametrItem.id}`,
          parametrItem
        )
        .then((res: any) => {
          this.calculateTotalPrice();
        });
    },
    calculateTotalPrice() {
      this.calculateModel.totalPrice = 0;
      this.calculateModel.buyingTotalPrice = 0;
      for (let index = 0; index < this.shoppingList.length; index++) {
        const element = this.shoppingList[index];
        this.calculateModel.totalPrice += Number(element.price);
        if (element.isBuy == true) {
          this.calculateModel.buyingTotalPrice += Number(element.price);
        }
        this.calculateModel.notBuyingTotalPrice =
          this.calculateModel.totalPrice - this.calculateModel.buyingTotalPrice;
      }
    },
  },
  mounted() {
    this.getShoppingList();
  },
  computed: {
    notBuyingLength() {
      //"Birinci yöntem:"
      // let count = 0;
      // for (let index = 0; index < this.shoppingList.length; index++) {
      //   const element = this.shoppingList[index];
      //   if (element.isBuy == false) {
      //     count++;
      //   }
      // }
      // return count;

      //"İkinci yöntem:"
      let array = this.shoppingList.filter((item) => item.isBuy == false);
      return array.length;
    },
  },
});
</script>

<style>
</style>
