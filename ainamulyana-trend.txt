(function () {
  "use strict";

  /* ==== Ambil VARIABEL DARI TEMPLATE ==== */
  if (typeof postperpage === "undefined" ||
      typeof numshowpage === "undefined" ||
      typeof upPageWord === "undefined" ||
      typeof downPageWord === "undefined") {
    console.warn("Pagination: variabel template belum lengkap");
    return;
  }

  var PER_PAGE = postperpage;
  var SHOW_PAGE = numshowpage;
  var PREV_TEXT = upPageWord;
  var NEXT_TEXT = downPageWord;

  var pager =
    document.getElementById("blog-pager") ||
    document.querySelector("[name='pageArea']") ||
    document.getElementById("pagination");

  if (!pager) return;

  function getCurrentPage() {
    var m = location.hash.match(/PageNo=(\d+)/);
    return m ? parseInt(m[1], 10) : 1;
  }

  function getLabel() {
    var m = location.pathname.match(/\/search\/label\/([^/?]+)/);
    return m ? decodeURIComponent(m[1]) : null;
  }

  function getTotalPosts(label) {
    var url = label
      ? "/feeds/posts/summary/-/" + label + "?alt=json&max-results=0"
      : "/feeds/posts/summary?alt=json&max-results=0";

    return fetch(url)
      .then(function (r) { return r.json(); })
      .then(function (j) {
        return parseInt(j.feed.openSearch$totalResults.$t, 10);
      });
  }

  function pageLink(page, label) {
    if (page === 1) {
      return label
        ? "/search/label/" + label + "?max-results=" + PER_PAGE
        : "/?max-results=" + PER_PAGE;
    }
    return "#PageNo=" + page;
  }

  function render(current, total, label) {
    var html = "<span class='page-info'>Page " + current + " of " + total + "</span>";

    if (current > 1) {
      html += "<a href='" + pageLink(current - 1, label) + "' rel='prev'>" + PREV_TEXT + "</a>";
    }

    var start = Math.max(
