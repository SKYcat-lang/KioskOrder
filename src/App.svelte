<script>
  let useId = 0;
  let orderId = 0;
  let useUser = null; // 선택된 사용자를 저장하는 변수
  const tableNumber = 3;
  let showOrderPopup = false;
  let showOldOrderPopup = false;
  let showOldOrderPopupData = [];

  class User {
    id = 0;
    cart = [];
  }

  class Item {
    name = "";
    image = "";
    desc = "";
    price = 0;
    constructor(name = "", image = "", desc = "", price = 0) {
      this.name = name;
      this.image = image;
      this.desc = desc;
      this.price = price;
    }
  }

  let userArray = [];
  let itemArray = [];
  let orderList = [];
  let orderTempData = [];

  itemArray = [
    new Item("Item A", "img", "desc", 9000),
    new Item("Item B", "img", "desc", 12000),
    new Item("Item C", "img", "desc", 5000),
    new Item("Item D", "img", "desc", 11000),
    new Item("Item E", "img", "desc", 8000)
  ];

  async function handleAddUser() {
    let tempUser = new User();
    tempUser.id = useId++;
    userArray = [...userArray, tempUser];
  }

  // 사용자 버튼 클릭 시 해당 사용자를 선택합니다.
  function handleSelectUser(user) {
    useUser = user;
  }

  function handleDeleteUser(user) {
    // 삭제 버튼 클릭 시 이벤트 버블링 방지
    userArray = userArray.filter(u => u.id !== user.id);
    if (useUser && useUser.id === user.id) {
      useUser = null;
    }
  }

  // 아이템이 cart에 없으면 추가
  function handleOrderItem(item) {
    if (!useUser) {
      alert("먼저 사용자를 선택해주세요.");
      return;
    }
    const index = userArray.findIndex(u => u.id === useUser.id);
    if (index !== -1) {
      // cart에서 해당 아이템을 찾기 (이름으로 비교)
      const exists = userArray[index].cart.find(i => i.name === item.name);
      if (!exists) {
        // 새 아이템 추가 시 수량 1 설정
        const newItem = { ...item, quantity: 1 };
        userArray[index].cart = [...userArray[index].cart, newItem];
        // 반응성을 위해 재할당
        useUser = userArray[index];
        userArray = [...userArray];
      }
    }
  }

  // 수량 증가
  function handleIncreaseQuantity(item) {
    if (!useUser) return;
    const index = userArray.findIndex(u => u.id === useUser.id);
    if (index !== -1) {
      const cartItem = userArray[index].cart.find(i => i.name === item.name);
      if (cartItem) {
        cartItem.quantity++;
        useUser = userArray[index];
        userArray = [...userArray];
      }
    }
  }

  // 수량 감소 (수량이 1이면 cart에서 제거)
  function handleDecreaseQuantity(item) {
    if (!useUser) return;
    const index = userArray.findIndex(u => u.id === useUser.id);
    if (index !== -1) {
      const cartIndex = userArray[index].cart.findIndex(i => i.name === item.name);
      if (cartIndex !== -1) {
        if (userArray[index].cart[cartIndex].quantity > 1) {
          userArray[index].cart[cartIndex].quantity--;
        } else {
          // 수량이 1이면 제거
          userArray[index].cart.splice(cartIndex, 1);
        }
        useUser = userArray[index];
        userArray = [...userArray];
      }
    }
  }

  function orderCompleted() {
    orderTempData = [];
    for (const user of userArray) {
      let orderObj = { id: user.id, cart: [], price: 0 };
      if (user.cart.length) {
        for (const cart of user.cart) {
          orderObj.cart.push(`${cart.name} (${cart.quantity}) : ${cart.quantity * cart.price}원`);
          orderObj.price += cart.quantity * cart.price;
        }
        orderTempData = [...orderTempData, orderObj];
      }
    }
    showOrderPopup = true;
  }

  function orderCompletedEnd() {
    if (orderTempData.length) {
      orderList = [...orderList, orderTempData];
      for (const user of userArray) {
        user.cart = [];
      }
      userArray = [...userArray];
      useUser.cart = [...useUser.cart];
      console.log(orderList);
      orderTempData = [];
      orderId++;
    }
    showOrderPopup = false;
  }

  function orderPrice(orders) {
    let price = 0;
    for (const user of orders) {
      price+=user.price;
    }
    return price;
  }

  function orderPriceAll() {
    let allPrice = 0;
    for (const order of orderList) {
      for (const user of order) {
        allPrice+=user.price;
      }
    }
    return allPrice;
  }

  function oldOrderShow(order) {
    showOldOrderPopupData = order;
    showOldOrderPopup = true;
  }
</script>

<main class="flex">
  <!-- 사이드바 영역 -->
  <aside class="fixed flex flex-col flex-shrink-0 w-64 h-screen bg-gray-200 p-2 rounded-md overflow-y-auto">
    <button
      on:click={handleAddUser}
      class="w-full bg-rose-500 hover:bg-rose-900 text-white py-2 px-4 rounded-md"
    >
      사용자 추가
    </button>
    {#if userArray.length}
      <div class="text-center font-medium text-gray-700 mt-4 space-y-2">
        {#each userArray as user (user.id)}
          <div class="flex items-center space-x-2">
            <button
              on:click|stopPropagation={() => handleDeleteUser(user)}
              class="p-1 text-xs text-white bg-gray-500 hover:bg-gray-700 rounded"
            >
              <!-- 삭제 아이콘 -->
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-trash3-fill" viewBox="0 0 16 16">
                <path d="M11 1.5v1h3.5a.5.5 0 0 1 0 1h-.538l-.853 10.66A2 2 0 0 1 11.115 16h-6.23a2 2 0 0 1-1.994-1.84L2.038 3.5H1.5a.5.5 0 0 1 0-1H5v-1A1.5 1.5 0 0 1 6.5 0h3A1.5 1.5 0 0 1 11 1.5m-5 0v1h4v-1a.5.5 0 0 0-.5-.5h-3a.5.5 0 0 0-.5.5M4.5 5.029l.5 8.5a.5.5 0 1 0 .998-.06l-.5-8.5a.5.5 0 1 0-.998.06m6.53-.528a.5.5 0 0 0-.528.47l-.5 8.5a.5.5 0 0 0 .998.058l.5-8.5a.5.5 0 0 0-.47-.528M8 4.5a.5.5 0 0 0-.5.5v8.5a.5.5 0 0 0 1 0V5a.5.5 0 0 0-.5-.5"/>
              </svg>
            </button>
            <button
              on:click={() => handleSelectUser(user)}
              class="flex-grow py-2 px-4 rounded-md {useUser && useUser.id === user.id ? 'hover:bg-blue-400 bg-blue-300' : 'hover:bg-gray-300 bg-gray-100'}"
            >
              <div class="font-bold text-center">사용자 {user.id}</div>
              <div class="text-left">
                주문:
                {#each user.cart as cart}
                  <div>{cart.name} ({cart.quantity}) : {cart.quantity * cart.price}원</div>
                {/each}
              </div>
            </button>
          </div>
        {/each}
      </div>
    {/if}
  </aside>
  <aside class="fixed right-0 flex flex-col flex-shrink-0 w-64 h-screen bg-gray-200 p-2 rounded-md overflow-y-auto">
    <!-- 테이블 번호 표시 -->
    <div class="text-center font-bold text-xl mb-4">
      테이블 번호: {tableNumber}
    </div>
    <div class="text-center">주문 내역</div>
    <div class="flex-grow text-center text-gray-700 mt-4 space-y-2 overflow-y-auto">
      {#if orderList.length}
        {#each orderList as orders, orderIndex}
          <button
            on:click={() => oldOrderShow(orders)}
            class="bg-gray-100 p-4 rounded-md hover:bg-gray-300"
          >
            <!-- 주문 번호: 가운데 정렬, 볼드체 -->
            <p class="text-center">주문 {orderIndex + 1}</p>
            <!-- 주문 내역: 아래에 나열 -->
            <div class="border-b mt-2 p-3">
              {#each orders as userOrder}
                <div>사용자 {userOrder.id} : {userOrder.price}원</div>
              {/each}
            </div>
            <div class="text-center font-bold">
              합계: {orderPrice(orders)}
            </div>
          </button>
        {/each}
      {/if}
    </div>
    <!-- 주문하기 버튼 -->
    <div class="flex justify-between bg-gray-200 p-2 z-10 text-xl mb-4">
      <span>총:</span>
      <span class="font-bold">{orderPriceAll()}원</span>
    </div>
    <div class="bg-gray-200 p-2 z-10">
      <button
        on:click={orderCompleted}
        class="w-full bg-green-500 hover:bg-green-700 text-white py-3 px-6 rounded-md text-lg"
      >
        주문하기
      </button>
    </div>
  </aside>
  
  <!-- 아이템 나열 영역 --><!-- 기존 컨테이너를 flex 컨테이너로 감싸고, 내부에 중앙 정렬 div 추가 -->
  <div class="mx-auto p-4 flex-1 flex justify-center">
    <div class="w-full max-w-6xl">
      <!-- 상품 목록 콘텐츠 (grid) -->
      <h2 class="text-2xl font-bold mb-4 text-center">상품 목록</h2>
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        {#each itemArray as item}
          {#if useUser && useUser.cart.find(i => i.name === item.name)}
            <!-- 이미 cart에 있는 아이템: 수량 조절 컨트롤 -->
            <div class="bg-white border border-gray-300 rounded-md p-4 flex flex-col items-center">
              <div class="font-semibold">{item.name}</div>
              <div class="text-sm text-gray-600">${item.price}</div>
              <div class="flex items-center mt-2">
                <button
                  on:click={() => handleDecreaseQuantity(item)}
                  class="px-2 py-1 bg-gray-300 rounded hover:bg-gray-400"
                >-</button>
                <!-- 수량 표시 span에 고정 너비와 중앙 정렬 클래스 추가 -->
                <span class="mx-2 w-10 text-center">
                  {useUser.cart.find(i => i.name === item.name).quantity}
                </span>
                <button
                  on:click={() => handleIncreaseQuantity(item)}
                  class="px-2 py-1 bg-gray-300 rounded hover:bg-gray-400"
                >+</button>
              </div>
            </div>
          {:else}
            <!-- 아직 cart에 없는 아이템: 추가 버튼 -->
            <button
              on:click={() => handleOrderItem(item)}
              class="bg-white border border-gray-300 rounded-md p-4 flex flex-col items-center hover:bg-gray-100"
            >
              <div class="font-semibold">{item.name}</div>
              <div class="text-sm text-gray-600">${item.price}</div>
            </button>
          {/if}
        {/each}
      </div>
    </div>
  </div>
  {#if showOrderPopup}
    <div class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">
      <div class="bg-white p-4 rounded-md max-w-md w-full">
        <h2 class="text-xl font-bold mb-2">주문 확인</h2>
        {#each orderTempData as order}
          <div class="border-b py-2">
            <p>사용자 {order.id}</p>
            {#each order.cart as item}
              <p>{item}</p>
            {/each}
            <p class="font-bold">금액: {order.price}원</p>
          </div>
        {/each}
        <p class="font-bold">총: {orderPrice(orderTempData)}원</p>
        <button on:click={orderCompletedEnd} class="mt-4 w-full bg-blue-500 hover:bg-blue-700 text-white py-2 rounded">
          주문하기
        </button>
        <button on:click={() => showOrderPopup = false} class="mt-4 w-full bg-blue-500 hover:bg-blue-700 text-white py-2 rounded">
          취소
        </button>
      </div>
    </div>
  {/if}
  
  {#if showOldOrderPopup}
    <div class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">
      <div class="bg-white p-4 rounded-md max-w-md w-full">
        <h2 class="text-xl font-bold mb-2">주문 확인</h2>
        {#each showOldOrderPopupData as user}
          <div class="border-b py-2">
            <p>사용자 {user.id}</p>
            {#each user.cart as item}
              <p>{item}</p>
            {/each}
            <p class="font-bold">금액: {user.price}원</p>
          </div>
        {/each}
        <p class="font-bold">총: {orderPrice(showOldOrderPopupData)}원</p>
        <button on:click={() => showOldOrderPopup = false} class="mt-4 w-full bg-blue-500 hover:bg-blue-700 text-white py-2 rounded">
          나가기
        </button>
      </div>
    </div>
  {/if}
</main>
