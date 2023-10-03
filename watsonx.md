---
layout: page
title: watsonx
permalink: /watsonx/
---

This is JJ's watsonx example page.

<script>
  window.watsonAssistantChatOptions = {
    integrationID: "be90fcd5-80ec-4e0d-b763-cae37ad865d7", // The ID of this integration.
    region: "us-south", // The region your integration is hosted in.
    serviceInstanceID: "75559e8c-2fb3-408e-aba2-63d99b78d887", // The ID of your service instance.
    onLoad: function(instance) { instance.render(); }
  };
  setTimeout(function(){
    const t=document.createElement('script');
    t.src="https://web-chat.global.assistant.watson.appdomain.cloud/versions/" + (window.watsonAssistantChatOptions.clientVersion || 'latest') + "/WatsonAssistantChatEntry.js";
    document.head.appendChild(t);
  });
</script>
