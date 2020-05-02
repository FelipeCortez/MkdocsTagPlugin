<input id="tag-search" style="background-color: #FEFEFE; border: 1px solid #979797; border-radius: 2px; font-size: 18px; color: #959595; width: 100%; padding: 10px; box-shadow: inset 2px 2px 0 rgba(0, 0, 0, .04), 2px 2px 0 rgba(0, 0, 0, .04); margin-bottom: 16px" />
<ul id="matching-tags" style="margin: 0; margin-bottom: 24px; display: flex"></ul>
<ul id="matching-docs" style="margin: 0"></ul>

<script>
 function filterByPred(tags, pred) {
   pred = pred.replace(/!/, " ! ");
   let tokens = pred.split(" ");
   tokens = tokens
     .filter(token => !!token.trim())
     .map(token => {
       if (["&&", "||", "!"].includes(token)) {
         return token;
       } else {
         try {
           return tags.some(tag => tag.match(token));
         } catch (e) {
           return true;
         }
       }
     });

   let expr = tokens.join(" ");

   try {
     return eval(expr);
   } catch (e) {
     return true;
   }
 }

 function renderDocList(docs, query) {
   let filteredDocs = docs.filter(doc => query.trim() == "" || filterByPred(doc.tags, query));

   let allTags = {};
   filteredDocs.forEach(doc => doc.tags.forEach(tag => allTags.hasOwnProperty(tag) ? allTags[tag] += 1 : allTags[tag] = 1));
   console.log(allTags);

   for (let [tag, qty] of Object.entries(allTags).sort((a, b) => b[1] - a[1])) {
     let tagLi = document.createElement("li");
     tagLi.style = "list-style-type: none; margin: 0; margin-right: 5px; margin-left: 0; border-radius: 4px; padding: 2px 4px; background-color: #F5F5F5";
     tagLi.textContent = `${tag}: ${qty}`;
     document.querySelector("#matching-tags").append(tagLi);
   }

   for (let doc of filteredDocs.sort((a, b) => a.title.localeCompare(b.title))) {
     let docLi = document.createElement("li");
     docLi.style = "list-style-type: none; margin-left: 0;";
     let docTags = document.createElement("span");
     for (let tag of doc.tags) {
       let tagEl = document.createElement("span");
       tagEl.innerHTML = tag;
       tagEl.style = "margin-left: 8px; border-radius: 4px; padding: 2px 4px; background-color: #F5F5F5";
       docTags.append(tagEl);
     }

     console.log(doc);
     docLi.innerHTML = `<a href="${doc.url}">${doc.title}</a>`;
     docLi.append(docTags);
     document.querySelector("#matching-docs").append(docLi);
   }
 }

 function killAllChildren(element) {
   let clone = element.cloneNode(false);
   element.parentNode.replaceChild(clone, element);
 }

 (async () => {
   await fetch("/tags.json")
     .then(r => r.text())
     .then(r => {
       let docs = JSON.parse(r);
       renderDocList(docs, "");

       document.querySelector("#tag-search").addEventListener("input", function (evt) {
         killAllChildren(document.querySelector("#matching-tags"));
         killAllChildren(document.querySelector("#matching-docs"));
         renderDocList(docs, this.value);
       });
     })
 })();
</script>
