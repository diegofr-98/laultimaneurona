---
title: Entendiendo NGRX
author: Diego Franco Roy
pubDatetime: 2024-10-31T23:00:00.000Z
postSlug: entendiendo-NGRX
featured: true
description: Aprende de forma sencilla el funcionamiento de NGRX mediante un ejemplo.
tags:
  - NGRX
  - Angular
ogImage: /og-image.jpeg
---

### ¿Qué es NGRX y cómo funciona?

NGRX es una biblioteca basada en Redux, implementada para Angular, que se enfoca en gestionar el estado de la aplicación de manera reactiva. Su arquitectura se basa en conceptos clave como acciones, reducers, efectos, store y selectors. El flujo de trabajo en NGRX es cíclico y reactivo: al realizar una acción, el estado se modifica y se notifica a los componentes que dependen de él, lo que permite que estos se actualicen automáticamente sin necesidad de manejar el estado directamente en cada uno.

Para entenderlo mejor, veamos cómo funciona cada elemento de NGRX:

* Acciones: Son simples objetos que describen un evento. Por ejemplo, “usuarioIniciaSesion” o “cargarListaDeProductos” son acciones que notifican cambios en la aplicación.
* Reducers: Funcionan como funciones puras que reciben una acción y el estado actual de la aplicación y retornan un nuevo estado, actualizando así la "Store".
* Store: Almacena el estado y permite que todos los componentes accedan a él a través de selectores.
* Selectors: Funciones que extraen datos específicos de la "Store" para facilitar su uso en los componentes.
* Effects: Permiten manejar operaciones asíncronas (como peticiones HTTP) de manera separada, interceptando las acciones para realizar, por ejemplo, llamadas a una API y luego disparar una acción con el resultado.

### ¿Cómo funciona el flujo de trabajo de NGRX?

Para visualizarlo mejor, aquí tienes una ilustración de cómo NGRX maneja el flujo de trabajo en Angular:

1\. El usuario realiza una acción en la interfaz (por ejemplo, quiere agregar un producto al carrito de compras).

2\. Esta acción desencadena una acción NGRX (por ejemplo, agregarProductoCarrito).

3\. La acción llega al reducer, donde se actualiza el estado del carrito en el store.

4\. Si es necesario, un effect intercepta esta acción, realiza una petición HTTP a una API y devuelve el resultado al store.

5\. Los componentes, que escuchan el estado a través de selectors, reciben automáticamente los cambios y se actualizan en la UI.

![](/NGRX-DIAGRAMA-FLUJO.svg)

### Ejemplo

Ahora vamos a ver con un [ejemplo](https://github.com/diegofr-98/cart-ngrx "Ejemplo Carrito Github") muy sencillo como  manejar el estado en lo que vendría a ser un carrito de compra.

### 1. Crear proyecto de Angular

```shell
ng new cart-ngrx
cd cart-ngrx
```

### 2. Instalar dependencias de NGRX

```shell
ng add @ngrx/store@latest
ng add @ngrx/effects@latest
ng add @ngrx/store-devtools@latest
```

### 3. Definir modelo de datos

Carpeta: src/app/models/product.model.ts

```typescript
export interface Product {
  id: number;
  name: string;
  price: number;
}

export interface CartItem {
  product: Product;
  quantity: number;
}
```

### 4. Definir estado inicial

Carpeta: src/app/state/cart.state.ts

```typescript
import { CartItem } from '../models/product.model';

export interface CartState {
  items: CartItem[];
}

export const initialCartState: CartState = {
  items: [
    {
      product: {
        id: 1,
        name: 'Product 1',
        price: 100
      },
      quantity: 1
    },
    {
      product: {
        id: 2,
        name: 'Product 2',
        price: 200
      },
      quantity: 2
    },
  ]
};
```

### 5. Definir acciones NGRX

Carpeta: src/app/state/cart.actions.ts

```typescript
import { createAction, props } from '@ngrx/store';
import { Product } from '../models/product.model';

export const addProduct = createAction(
  '[Cart] Add Product',
  props<{ product: Product }>()
);

export const removeProduct = createAction(
  '[Cart] Remove Product',
  props<{ productId: number }>()
);

export const clearCart = createAction('[Cart] Clear Cart');

```

### 6. Crear reducer

Carpeta: src/app/state/cart.reducer.ts

```typescript
import { createReducer, on } from '@ngrx/store';
import { addProduct, removeProduct, clearCart } from './cart.actions';
import { initialCartState } from './cart.state';

export const cartReducer = createReducer(
  initialCartState,
  on(addProduct, (state, { product }) => {
    const existingItem = state.items.find(item => item.product.id === product.id);
    const updatedItems = existingItem
      ? state.items.map(item =>
        item.product.id === product.id
          ? { ...item, quantity: item.quantity + 1 }
          : item
      )
      : [...state.items, { product, quantity: 1 }];

    return { ...state, items: updatedItems };
  }),
  on(removeProduct, (state, { productId }) => ({
    ...state,
    items: state.items.filter(item => item.product.id !== productId)
  })),
  on(clearCart, state => initialCartState),
  // on(clearCart, state => ({...state, items: []}))
);
```

### 7. Crear componente Cart

```shell
ng generate component cart
```

### 8. Agregar lógica al componente

```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { addProduct, removeProduct, clearCart } from '../state/cart.actions';
import { CartState } from '../state/cart.state';
import { Product, CartItem } from '../models/product.model';
import { AsyncPipe } from '@angular/common';

@Component({
  selector: 'app-cart',
  standalone: true,
  imports: [AsyncPipe],
  templateUrl: './cart.component.html',
  styleUrl: './cart.component.css'
})
export class CartComponent {
  cartItems$: Observable<CartItem[]>;
  constructor(private store: Store<{ cart: CartState }>) {
    this.cartItems$ = store.select(state => state.cart.items);
  }

  addProduct(product: Product) {
    this.store.dispatch(addProduct({ product }));
  }

  removeProduct(productId: number) {
    this.store.dispatch(removeProduct({ productId }));
  }

  clearCart() {
    this.store.dispatch(clearCart());
  }
}
```

### 9. Y por último el template

```html
@if (cartItems$ | async; as cartItems) {
  @for (item of cartItems; track item.product.id) {
    <p>{{ item.product.name }} - {{ item.quantity }} - ${{ item.product.price * item.quantity }}</p>
    <button (click)="addProduct(item.product)">Añadir</button>
    <button (click)="removeProduct(item.product.id)">Eliminar</button>
  }
}

<br>
<button (click)="clearCart()">Vaciar Carrito</button>
```

### ¿Cuándo Usar NGRX y Cuándo No?

NGRX es útil pero no siempre es necesaria. Usarla depende de la complejidad de la aplicación y de cuántos componentes requieran datos centralizados.

Cuándo usar NGRX:

* Cuando manejas una aplicación compleja y quieres un estado centralizado.
* Cuando múltiples componentes necesitan acceder y modificar datos compartidos.
* Cuando tienes un flujo de datos complejo con múltiples acciones y efectos asíncronos.

Cuándo evitar NGRX:

* En aplicaciones pequeñas con poco o nulo flujo compartido de datos.
* Si la aplicación es sencilla y el estado se puede gestionar localmente en los componentes.
* Cuando el tiempo y la complejidad que requiere aprender y configurar una solución es mayor que la mejora que aporta en la gestión del estado.
