<script>
  /* components */
  import Input from "$lib/components/Input.svelte";
  import Icons from "$lib/components/Icons.svelte";
  import Modal from "$lib/components/Modal.svelte";
  import TextArea from "$lib/components/TextArea.svelte";
  import Loader from "$lib/components/Loader.svelte";
  /* navigation */
  import { goto } from "$app/navigation";
  /* utils */
  import { dateToLocalString, timeToLocalString } from "$lib/utils/date.js";
  import { getCardBrand } from "$lib/utils/brands.js";
  import {
    successCustomMsgToast,
    errorCustomMsgToast,
  } from "$lib/utils/toast.js";
  import { copyLinkToClipboard } from "$lib/utils/copyToClipboard.js";
  import { currencyFormatLocal } from "$lib/utils/currencyFormatLocal";
  /* stores */
  import { previousPage } from "$lib/stores";
  /* svelte */
  import { onMount } from "svelte";
  import { error } from "@sveltejs/kit";
  /* client */
  import {
    axiosDevicesClient,
    ticketsClient,
    axiosFraudPreventionManagementJSON,
    profilesClient,
  } from "$lib/repos/axios";
  /* controllers */
  import { appErrorResponseHandler } from "$lib/handlers/error.handler";
  import {
    transactionStatus,
    transactionCancelValidation,
  } from "$lib/handlers/transaction-status.handler";

  export let data;
  let transaction = data?.response;
  let data3ds = data?.data3ds;
  let cardIcon = "";
  let modalClarification;
  let cancelModal;
  let refundModal;
  let modalCancel;
  let link;
  let loading = false;
  let transparent = false;
  let clarification = {
    transaction: transaction._id,
    description: "",
  };

  let cancelData = {
    amount: "",
    url: "",
    description: "",
    email: "",
  };

  let cancel = {
    amount: data?.response?.Amount,
    ["SIC Code"]: data?.response?.["SIC Code"],
    ["Application PAN"]: data?.response?.["Application PAN"],
    expirationDate: "",
    ["ID Transaction"]: data?.response?.["ID Transaction"],
    ["ID Afiliate"]: data?.response?.["ID Afiliate"],
    ["Afiliate Number"]: data?.response?.["Afiliate Number"],
    ["ID Aggregator"]: data?.response?.["ID Aggregator"],
    authorization: data?.response?.authorization,
    POS: data?.response?.POS,
    ["ID Terminal"]: data?.response?.["ID Terminal"],
    originalElements: data?.response?.originalElements,
    email: data?.response?.["Cardholder Email"],
    ["Cardholder Name"]: data?.response?.["Cardholder Name"],
    ["Cardholder Phone"]: data?.response?.["Cardholder Phone"],
    ["Transaction Date"]: data?.response?.["Transaction Date"],
    ["Transaction Time"]: data?.response?.["Transaction Time"],
    ["Points BBVA"]: data?.response?.["Points BBVA"],
    MSI: data?.response?.MSI,
    commerce: data?.response?.commerce,
    commerceName: data?.response?.commerceName ?? "",
    /* For the later version */
    description: "Link de Cancelación",
    transactionId: data?.response?.transaction,
  };

  /* let cancel = {
    description: "Link de Cancelación",
    transactionId: data?.response?.transaction
  } */

  const returnToPreviousPage = () => {
    history.back();
  };

  const showModal = (option) => {
    option.show();
  };
  const closeModal = (option) => {
    option.closeModal();
  };
  const handleClarification = async () => {
    try {
      const response = await ticketsClient.post(
        `/ticket/clarification/transaction`,
        clarification
      );
      successCustomMsgToast(`Tu ticket de aclaración se ha generado con éxito`);
      return { ...response.data?.response };
    } catch (err) {
      errorCustomMsgToast(`Ocurrió un error, intenta de nuevo`);
      const handler = await appErrorResponseHandler(err);
      const code = handler?.code ?? 500;
      const message = handler?.message ?? "¡Algo salió mal!";
      throw new error(code, message);
    }
  };

  const sendTransactionByEmail = async () => {
    try {
      const user = await profilesClient.get(`/user/profile`);
      const body = {
        address: user.address ?? undefined,
        scheme: getCardBrand(transaction["Application PAN"]),
        commerceName: user.businessName ?? undefined,
        authorization: transaction.authorization,
      };
      const response = await axiosDevicesClient.post(
        `/transaction/detail/${transaction._id}/email`,
        {
          body,
        }
      );
      successCustomMsgToast(
        `Correo enviado con éxito a tu dirección asociada a Lkl Pay`
      );
      return { ...response.data?.response };
    } catch (err) {
      errorCustomMsgToast(`Ocurrió un error, intenta de nuevo`);
      const handler = await appErrorResponseHandler(err);
      const code = handler?.code ?? 500;
      const message = handler?.message ?? "¡Algo salió mal!";
      throw new error(code, message);
    }
  };

  /* Función cancelar transacción */
  const cancelTransaction = async () => {
    cancelModal.closeModal()
    loading = true;
    try {
      const response = await axiosFraudPreventionManagementJSON.post(
        `/link/cancel`,
        cancel
      );
      cancelData = {
        amount: response?.data?.response?.amount,
        url: response?.data?.response?.url,
        description: response?.data?.response?.description,
        email: response?.data?.response?.email,
      };
      // formSuccess(link);
      modalCancel.show();
    } catch (err) {
      errorCustomMsgToast(`Ocurrió un error, intenta de nuevo`);
      const handler = await appErrorResponseHandler(err);
      const code = handler?.code ?? 500;
      const message = handler?.message ?? "¡Algo salió mal!";
      throw new error(code, message);
    } finally {
      loading = false;
    }
  };

  const getPercentage = (total, commission) => {
    return ((commission * 100) / total).toFixed(2);
  };

  const refundTransaction = async () => {
    console.log(data.response.transaction)
    loading = true;
    try {
      const response = await axiosFraudPreventionManagementJSON.post(
        `/e/refund`, {
          transactionId: data.response.transaction,
          // card: data.response["Application PAN"],
        }
      );
      refundModal.closeModal();
      successCustomMsgToast(`La devolución se realizó con éxito`);
    } catch (err) {
      refundModal.closeModal();
      errorCustomMsgToast(`Ocurrió un error, intenta de nuevo`);
      const handler = await appErrorResponseHandler(err);
      const code = handler?.code ?? 500;
      const message = handler?.message ?? "¡Algo salió mal!";
      throw new error(code, message);
    } finally {
      loading = false;
    }
  }

  const removeBackdrop = async () => {
    transparent = true;
    setTimeout(() => {
      transparent = false;
    }, 3000);
  };
</script>

<Modal className={`modal-medium`} bind:this={modalClarification}>
  <div slot="header">
    <p>Solicitar Aclaración</p>
  </div>
  <div slot="content">
    <div class="clarifications">
      <div class="title">Recibo N°</div>
      <div class="description">
        <p>{clarification.transaction}</p>
      </div>
      <!-- <Select bind:optionsList={clarificationsList} defaultText={"Elige una opción"} label="Tipo de Aclaración" id="clarificationType" bind:value={clarification.type}/> -->
    </div>
    <div class="clarification-description">
      <TextArea
        bind:value={clarification.description}
        label="Descripción"
        placeholder="¿Qué problema hay con esta transacción?"
        id="clarificationDescription"
        name="clarificationDescription"
      />
    </div>
  </div>
  <div class="modal-buttons" slot="footer">
    <Input
      on:click={() => handleClarification()}
      on:click={closeModal(modalClarification)}
      label="Enviar Aclaración"
      id="buttonSaveModalClarification"
      type="button"
      className={`
        ${clarification.description != "" ? "btn" : "btn-plain disabled"}`}
      icon=""
    />
  </div>
</Modal>

<!-- Modal Cancel -->
<Modal
  id="modalCancelLinkData"
  bind:transparent
  className={`modal-small`}
  bind:this={modalCancel}
>
  <div slot="header">
    <div class="svg">
      <p>Datos de Cancelación</p>
      <span><Icons name="success-circle" width="24" height="24" /></span>
    </div>
  </div>
  <div slot="content">
    <div class="thin-divider" />
    <div class="modal-content">
      <div class="column-element">
        <span class="copy-link">
          Enlace
          <div class="copy-link__icon">
            <label for="copy">
              <Icons name="file-copy" width="16" height="16" />
            </label>
            <input
              type="button"
              id="copy"
              name="copy"
              on:click={copyLinkToClipboard(link, "modalCancelLinkData")}
            />
          </div>
        </span>
        <textarea readonly bind:this={link} id="link" name="link"
          >{cancelData.url}</textarea
        >
      </div>
      <div class="column-element">
        <span>Monto</span>
        <p>{currencyFormatLocal(cancelData.amount)}</p>
      </div>
      <div class="column-element">
        <span>Concepto</span>
        <p>{cancelData.description}</p>
      </div>
      <div class="column-element">
        <span>E-Mail</span>
        <p>{cancelData.email}</p>
      </div>
    </div>
  </div>
  <div class="modal-buttons" slot="footer">
    <Input
      on:click={closeModal(modalCancel)}
      label="Cerrar"
      id="buttonCloseModalImmediateDepositPreference"
      type="button"
      className="btn-success"
      icon=""
    />
  </div>
</Modal>

<!-- Modal Confirm Cancel -->
<Modal
  id="modalConfirmCancel"
  bind:transparent
  className={`modal-small`}
  bind:this={cancelModal}
>
  <div slot="header">
    <div class="svg">
      <p>Cancelación</p>
    </div>
  </div>
  <div slot="content">
    <div class="thin-divider" />
    <div class="modal-content">
      <div class="column-element">
      </div>
      <div class="column-element">
        <p style="text-align:center;">¿Deseas solicitar una cancelación de la transacción?</p>
      </div>
    </div>
  </div>
  <div class="modal-buttons" slot="footer">
    <Input
      on:click={cancelTransaction}
      label="Sí, aceptar"
      id="buttonAcceptModalConfirmCancel"
      type="button"
      className="btn-success"
      icon=""
    />
    <Input
      on:click={cancelModal.closeModal()}
      label="No, cerrar"
      id="buttonCloseModalConfirmCancel"
      type="button"
      className="border-btn-error"
      icon=""
    />
  </div>
</Modal>
<!-- Modal Confirm Refund -->
<Modal
  id="modalConfirmRefund"
  bind:transparent
  className={`modal-small`}
  bind:this={refundModal}
>
  <div slot="header">
    <div class="svg">
      <p>Devolución</p>
    </div>
  </div>
  <div slot="content">
    <div class="thin-divider" />
    <div class="modal-content">
      <div class="column-element">
      </div>
      <div class="column-element">
        <p style="text-align:center;">¿Deseas solicitar una devolución de la transacción?</p>
      </div>
    </div>
  </div>
  <div class="modal-buttons" slot="footer">
    <Input
      on:click={refundTransaction}
      label="Sí, aceptar"
      id="buttonAcceptModalConfirmRefund"
      type="button"
      className="btn-success"
      icon=""
    />
    <Input
      on:click={refundModal.closeModal()}
      label="No, cerrar"
      id="buttonCloseModalConfirmRefund"
      type="button"
      className="border-btn-error"
      icon=""
    />
  </div>
</Modal>

{#if loading}
  <Loader />
{:else}
  <div class="return no-print">
    <Input
      on:click={returnToPreviousPage}
      label="Regresar"
      id="detailsReturnButton"
      type="button"
      className="btn-plain"
      icon=""
    />
  </div>
  <div class="transaction-details">
    <div class="details__top">
      <b>Recibo #{transaction["ID Transaction"]}</b>
      <p>
        {dateToLocalString(transaction["Transaction Date"])}
        {timeToLocalString(transaction["Transaction Time"])}
      </p>
    </div>
    <div class="details__middle">
      <div class="details-left responsive">
        <div class="section">
          <div class="title">Datos</div>
          {#if transaction.orderId}
            <div class="item">
              <b>ID Orden</b>
              <p>{transaction.orderId ?? "N/A"}</p>
            </div>
          {/if}
          <div class="item">
            <b>Referencia</b>
            <p>{transaction.reference ?? "N/A"}</p>
          </div>
          <div class="item">
            <b>TVR</b>
            <p>{transaction["ID Terminal"] ?? "N/A"}</p>
          </div>
          <div class="item">
            <b>AID</b>
            <p>{transaction["Terminal Capabilities"] ?? "N/A"}</p>
          </div>
          <div class="item">
            <b>Autorización</b>
            <p>{transaction.authorization ?? "N/A"}</p>
          </div>
          <div class="item">
            <b>Tipo de Tarjeta</b>
            <p>{getCardBrand(transaction["Application PAN"]) ?? "N/A"}</p>
          </div>
          <div class="item">
            <b>Medio de Pago</b>
            <p>
              <i class="icon tooltip">
                <Icons
                  name={transaction.type === "tpv"
                    ? "terminal"
                    : transaction.type === "e-commerce"
                    ? "qr-code"
                    : "terminal"}
                  width="24"
                  height="24"
                />
                <span class="tooltiptext"
                  >{transaction.type === "tpv"
                    ? "Terminal Punto de Venta"
                    : transaction.type === "e-commerce"
                    ? "Link de Pago"
                    : ""}</span
                >
              </i>
            </p>
          </div>
        </div>
      </div>
      <div class="details-center">
        <div
          class={`details-card ${getCardBrand(
            transaction["Application PAN"]
          ).toLowerCase()}`}
        >
          <div class="details-card__top">
            <b>Detalle de Venta</b>
          </div>
          <div class="details-card__middle">
            <div class="item">
              <div class="item__title">
                <b>Tarjeta Utilizada</b>
              </div>
              <div class="item__content first">
                <p>
                  <span
                    >{"**** **** **** " +
                      transaction["Application PAN"].substr(-4)}</span
                  >
                </p>
              </div>
            </div>
            <div class="item">
              <div class="item__title">
                <b>Tipo de Tarjeta</b>
              </div>
              <div class="item__content">
                <p>
                  <Icons
                    name={`${getCardBrand(
                      transaction["Application PAN"]
                    ).toLowerCase()}`}
                    width="24"
                    height="24"
                  />
                </p>
              </div>
            </div>
            <div class="item">
              <div class="item__title">
                <b>Total de la Venta</b>
              </div>
              <div class="item__content last">
                <p>
                  <span>
                    {currencyFormatLocal(transaction.Amount)}
                  </span>
                </p>
              </div>
            </div>
          </div>
          <div class="details-card__bottom">
            <div class="item">
              <div class="item__title">
                <b>Estatus</b>
              </div>
              <div class="item__content first">
                <p>{transactionStatus(transaction.transactionStatus)}</p>
              </div>
            </div>
            <div class="item">
              <div class="item__title">
                <b>Comisión</b>
              </div>
              <div class="item__content">
                <p>
                  {currencyFormatLocal(transaction.comission)}
                </p>
                <p>{`(${getPercentage(
                  transaction.Amount,
                  transaction.comission
                )}%)`}</p>
              </div>
            </div>
            <div class="item">
              <div class="item__title">
                <b>IVA</b>
              </div>
              <div class="item__content">
                <p>
                  {currencyFormatLocal(transaction.iva)}
                </p>
                <p>{`(16%)`}</p>
              </div>
              <span />
            </div>
            <div class="item">
              <div class="item__title">
                <b>Total a Depositar</b>
              </div>
              <div class="item__content last">
                <p>
                  {currencyFormatLocal(transaction.toDeposit)}
                </p>
              </div>
              <span />
            </div>
          </div>
        </div>
      </div>
      {#if data3ds != null}
        <div class="details-right responsive">
          <div class="section">
            <div class="title">Datos 3D Secure</div>
            <div class="item">
              <b>ID</b>
              <p>{data3ds.id ?? "N/A"}</p>
            </div>
            <div class="item">
              <b>ECI</b>
              <p>{data3ds.eci ?? "N/A"}</p>
            </div>
            <div class="item">
              <b>Token</b>
              <p>{data3ds.token ?? "N/A"}</p>
            </div>
            <div class="item">
              <b>ID Transacción 3DS</b>
              <p>{data3ds.threeDSServerTransactionId ?? "N/A"}</p>
            </div>
            <div class="item">
              <b>CAVV</b>
              <p>{data3ds.cavv ?? "N/A"}</p>
            </div>
            <div class="item">
              <b>Estatus</b>
              <p>{data3ds.status ?? "N/A"}</p>
            </div>
          </div>
        </div>
      {/if}
      <!-- <div class="details-right no-print responsive">
        <div class="title">Reportes</div>
        <div class="export-buttons">
          <Input
            label=""
            id="csv-export"
            type="button"
            className="btn-plain fill-blue btn-square "
            icon="csv-fill"
          />
          <Input
            label=""
            id="excel-export"
            type="button"
            className="btn-plain fill-green btn-square "
            icon="xls-fill"
          />
          <Input
            label=""
            id="pdf-export"
            type=""
            className="btn-plain fill-red btn-square "
            icon="pdf-fill"
          />
        </div>
      </div> -->
    </div>
    <div class="details__bottom">
      <div class="card-buttons no-print">
        {#if transactionCancelValidation(transaction.transactionStatus, transaction["Transaction Date"], transaction.type)}
          <div class="reverse-button">
            <Input
              on:click={cancelModal.show()}
              label="Cancelar"
              id="reverseTransaction"
              type="button"
              className="border-btn-error"
              icon=""
            />
          </div>
        {/if}
          <div class="reverse-button">
            <Input
              on:click={refundModal.show()}
              label="Devolución"
              id="refundTransaction"
              type="button"
              className="border-btn-error"
              icon=""
            />
          </div>
        <!-- <div class="clarification-button">
          <Input
            on:click={showModal(modalClarification)}
            label="Aclaración"
            id="transactionClarification"
            type="button"
            className="btn-plain"
            icon=""
          />
        </div>
        <div class="email-button">
          <Input
            on:click={sendTransactionByEmail}
            label="Enviar por e-mail"
            id="emailTransaction"
            type="button"
            className="btn-plain"
            icon=""
          />
        </div>
        <div class="print-button">
          <Input
            on:click={() => window.print()}
            label="Imprimir Recibo"
            id="printTransaction"
            type="button"
            className="btn-plain"
            icon=""
          />
        </div> -->
      </div>
    </div>
  </div>
{/if}

<style lang="scss">
  @import "src/lib/styles/transactions/detail.scss";

  .svg {
    display: flex;
    align-items: center;
    gap: 0.25rem;
    span {
      color: $secondary-dark;
    }
  }

  .thin-divider {
    width: 100%;
    border: 1px solid $grey;
    margin: 0 0 10px 0;
  }

  .modal-content {
    display: flex;
    flex-direction: column;
    .column-element {
      display: flex;
      flex-direction: column;
      margin: 0 0 10px 0;
      span {
        font-weight: 600;
        color: $primary-dark;
        font-size: 1.125rem;
        &.copy-link {
          display: flex;
        }
        .copy-link__icon {
          color: $primary-dark;
          label {
            cursor: pointer;
          }
          input {
            display: none;
          }
          &:hover {
            color: $primary-light;
          }
        }
      }
      p,
      textarea {
        font-weight: 400;
        color: $primary-dark;
        font-size: 0.875;
        word-wrap: break-word;
      }

      textarea {
        resize: none;
        border: none;
        outline: none;
        overflow: hidden;
        height: 3rem;

        &::selection {
          color: $primary-light;
          background: transparent;
        }
      }
    }
  }
  .details__bottom {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    .card-buttons {
      width: 100%;
      height: 2.5rem; /* 40px */
      display: flex;
      margin-top: 2rem; /* 32px */
      gap: 1rem; /* 16px */
      justify-content: center;
      .reverse-button {
        display: flex;
        width: 6rem; /* 80px */
      }
      .clarification-button {
        display: flex;
        width: 6rem; /* 80px */
      }
      .email-button {
        display: flex;
        width: 7.5rem; /* 120px */
      }
      .print-button {
        display: flex;
        width: 7.5rem; /* 120px */
      }
    }
  }

  .icon {
    color: $grey;
    &.tooltip {
      position: relative;
      font-weight: 500;
      font-size: 0.8125rem;
      line-height: 1.25rem;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      text-decoration: none;
      .tooltiptext {
        visibility: hidden;
        width: fit-content;
        background-color: #555;
        color: #fff;
        text-align: center;
        border-radius: 6px;
        padding: 5px 5px;
        position: absolute;
        z-index: 1;
        bottom: 125%;
        left: 280%;
        margin-left: -100px;
        opacity: 0;
        transition: opacity 0.3s;
      }

      .tooltiptext::after {
        content: "";
        position: absolute;
        top: 100%;
        left: 50%;
        margin-left: -5px;
        border-width: 5px;
        border-style: solid;
        border-color: #555 transparent transparent transparent;
      }
      &:hover .tooltiptext {
        visibility: visible;
        opacity: 1;
      }
    }
  }
</style>
