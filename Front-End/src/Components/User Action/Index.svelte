<script>
  import Table from "./Table.svelte";

  import toast, { Toaster } from "svelte-french-toast";
  import UserForm from "../User Action/UserForm.svelte";
  import Header from "../Shared/Header/Header.svelte";
  let foundSearchData = [];
  let userData = [];
  let dataToBeUpdated = "";
  let totalPages = 0;
  let totalRecords = 0;
  let loading = true;
  $: page = 1;

  let searchData = "";

  let totalRecordPerPage = "";

  $: block = "dashboard";
  $: buttonStatusOnEvent = (event) => {
    if (event.detail.block === "userForm") {
      block = "userForm";
    } else if (event.detail.block === "dashboard") {
      page = 1;
      block = "dashboard";
    } else if (event.detail.block === "searchField") {
      block = "searchField";
      searchData = event.detail.data;

      getSingleId(searchData);
    }
  };
  $: pageNumber = (e) => {
    if (e.detail.message === "next" && page < totalPages) {
      page++;

      fetchData();
    } else if (e.detail.message === "prev" && page > 1 && page <= totalPages) {
      page--;
      fetchData();
    } else if (e.detail > 0 && e.detail <= totalPages) {
      page = e.detail;
      fetchData();
    }
  };

  const fetchData = async () => {
    try {
      const url = `http://localhost:4000/user/?page=${page}`;

      const res = await fetch(url, {
        method: "GET",
      });
      let response = await res.json();

      userData = response.data;
      console.log(userData);
      totalPages = response.totalPages; // Total number of pages (totalrecord/8)
      totalRecords = response.totalRecords; //Total record in the db
      totalRecordPerPage = userData.length; //Records per page
    } catch (error) {
      console.log(error);
    } finally {
      () => {
        loading = false;
      };
    }
  };

  var deleteClick = async (e) => {
    try {
      const res = await fetch("http://localhost:4000/user/" + e.detail, {
        method: "DELETE",
      });
      const response = await res.text();

      var delId = "";
      for (let i = 0; i < userData.length; i++) {
        if (userData[i].id === e.detail) {
          delId = i;
        }
      }
      userData.splice(delId, 1);
      userData = userData;

      toast.success(`User Data Deleted Successfully`, {
        position: "bottom-center",
      });
      fetchData();
    } catch (error) {
      console.log(error);
    }
  };

  var updateDataSend = async (e) => {
    try {
      block = "updateUser";
      dataToBeUpdated = e.detail;
    } catch (error) {
      console.log(error);
    }
  };
  var updatingTheData = async (e) => {
    var updatedData = e.detail;
    try {
      if (
        updatedData.f_name.trim() === "" ||
        updatedData.m_name.trim() === "" ||
        updatedData.l_name.trim() === ""
      ) {
        toast.error(`Name fields cannot be blank`, {
          position: "bottom-center",
        });
      } else {
        const res = await fetch(
          "http://localhost:4000/user/" + updatedData.id,
          {
            method: "PATCH",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              f_name: updatedData.f_name,
              m_name: updatedData.m_name,
              l_name: updatedData.l_name,
              email: updatedData.email,
              contact: updatedData.contact,
              password: updatedData.password,
              confirmPassword: updatedData.confirmPassword,
              date_of_birth: updatedData.date_of_birth,
              gender: updatedData.gender,
              address_line1: updatedData.address_line1,
              address_line2: updatedData.address_line2,
              landmark: updatedData.landmark,
              city_name: dataToBeUpdated.city_name,
              zip_code: updatedData.zip_code,
              state_name: updatedData.state_name,
              passForVerification: updatedData.passForVerification,
              city_id: updatedData.city_id,
              add_id: updatedData.add_id,
              state_id: updatedData.state_id,
            }),
          }
        );
        const response = await res;
        var resText = await response.text();
        console.log(resText);
        if (response.status === 200) {
          block = "dashboard";
          toast.success(`Data Updated For ${updatedData.f_name}`, {
            position: "bottom-center",
          });
          await fetchData();
        } else {
          toast.error(`${resText}`, {
            position: "bottom-center",
          });
        }
      }
    } catch (error) {
      console.log(error);
    }
  };
  const getSingleId = async (email) => {
    try {
      const url = `http://localhost:4000/user/${email}`;

      const res = await fetch(url, {
        method: "GET",
      });
      let response = await res.json();
      console.log(response);
      // foundSearchData = foundSearchData.push(response);
      foundSearchData = [];
      for (let index = 0; index < response.length; index++) {
        foundSearchData[index] = response[index];
      }

      totalPages = 1; // Total number of pages (totalrecord/8)
      totalRecords = response.length; //Total record in the db
      totalRecordPerPage = response.length; //Records per page
    } catch (error) {
      console.log(error.text);
      toast.error(`Data Not Found`, {
        position: "bottom-center",
      });
      block = "dashboard";
    }
  };
  const doPost = async (e) => {
    try {
      let dataToBeAdded = e.detail;
      if (dataToBeAdded.confirmPassword !== dataToBeAdded.password) {
        dataToBeAdded.password = "";
      }
      const res = await fetch("http://localhost:4000/user/", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          f_name: dataToBeAdded.f_name,
          m_name: dataToBeAdded.m_name,
          l_name: dataToBeAdded.l_name,
          email: dataToBeAdded.email,
          contact: dataToBeAdded.contact,
          password: dataToBeAdded.password,
          date_of_birth: dataToBeAdded.date_of_birth,
          gender: dataToBeAdded.gender,
          address_line1: dataToBeAdded.address_line1,
          address_line2: dataToBeAdded.address_line2,
          landmark: dataToBeAdded.landmark,
          city_name: dataToBeAdded.city_name,
          zip_code: dataToBeAdded.zip_code,
          state_name: dataToBeAdded.state_name,
        }),
      });
      const response = await res.text();
      const status = await res.status;

      if (status === 200) {
        block = "dashboard";
        toast.success(`${response}`, {
          position: "bottom-center",
        });
        fetchData();
        page = totalPages;
      } else {
        toast.error(`${response}`, {
          position: "bottom-center",
        });
      }
    } catch (error) {
      console.log(error);
    }
  };
</script>

<Toaster />
<Header {block} on:message={buttonStatusOnEvent} />
{#if block === "dashboard"}
  <Table
    {userData}
    {fetchData}
    {totalRecords}
    {totalPages}
    {totalRecordPerPage}
    {page}
    on:delete={deleteClick}
    on:update={updateDataSend}
    on:page={pageNumber}
    on:next={pageNumber}
    on:prev={pageNumber}
  />
{:else if block === "userForm"}
  <UserForm on:post={doPost} />
{:else if block === "updateUser"}
  <UserForm {dataToBeUpdated} on:update={updatingTheData} />
{:else if block === "searchField"}
  <Table
    userData={foundSearchData}
    fetchData={getSingleId}
    {totalRecords}
    {totalPages}
    {searchData}
    {totalRecordPerPage}
    {page}
    on:delete={deleteClick}
    on:update={updateDataSend}
    on:page={pageNumber}
    on:next={pageNumber}
    on:prev={pageNumber}
  />
{/if}
