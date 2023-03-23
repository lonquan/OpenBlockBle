<script>
export default {}
</script>

<style lang="scss">
.h-full-screen {
  height: 100vh;
}

.page {
  background-image: linear-gradient(#3872FD, #F2F6FE);
}

.text-base {
  font-size: 16px;
  font-weight: 300;
}

.text-sm {
  font-size: 14px;
}

.tit {
  font-weight: 400;
  display: flex;
  justify-content: flex-start;
  align-items: center;

  .mark {
    display: inline-block;
    width: 4px;
    height: 18px;
    background: #3872FD;
    border-radius: 4px;
    margin-right: 4px;
  }
}

.card {
  background: #ffffff;
  border-radius: 10px;
  padding: 10px 20px;
  margin: 20px;
  box-shadow: 0px 0px 12px rgba(0, 0, 0, .12);

  .card-group {
    display: flex;
    justify-content: space-between;
    align-items: center;

    .btn {
      font-size: 14px;
      color: #3872FD;
      text-decoration: underline;
    }
  }
}

.empty {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px 10px;
}

.derives {
  padding-top: 15px;

  .item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid #eee;

    .btn {
      font-size: 12px;
      background: #3872FD;
      color: #ffffff;
      padding: 4px 10px;
      border-radius: 4px;
    }

    &:last-child {
      border-bottom: none;
    }
  }
}
</style>
