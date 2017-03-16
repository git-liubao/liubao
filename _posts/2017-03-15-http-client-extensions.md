---
layout: post
title: HttpClientExtensions in C#
category: Language
tags:
- C#
- HttpClient
---

## Keyboard shortcuts in Windows

{% highlight c# %}
#region HttpClientExtensions.cs

namespace Common
{
    using System;
    using System.Collections.Generic;
    using System.Diagnostics;
    using System.Net;
    using System.Net.Http;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;

    /// <summary>
    /// Defines extension methods for HttpClient.
    /// </summary>
    public static class HttpClientExtensions
    {
        /// <summary>
        /// The default maximum retry count
        /// </summary>
        public const int DefaultMaximumRetryCount = 3;

        /// <summary>
        /// The default retry interval milliseconds
        /// </summary>
        public const int DefaultRetryIntervalMilliseconds = 300;

        /// <summary>
        /// Initiate a http client.
        /// </summary>
        /// <param name="proxy">The proxy.</param>
        /// <param name="timeout">The timeout.</param>
        /// <param name="httpClientHandler">The http client handler.</param>
        /// <returns>The HttpClient.</returns>
        public static HttpClient InitiateHttpClient(string proxy = null, int timeout = 60000, HttpClientHandler httpClientHandler = null)
        {
            if (httpClientHandler == null)
            {
                httpClientHandler = new HttpClientHandler();
                if (!string.IsNullOrWhiteSpace(proxy))
                {
                    httpClientHandler.Proxy = new WebProxy(new Uri(proxy));
                }
            }

            var httpClient = new HttpClient(httpClientHandler, true);
            httpClient.Timeout = TimeSpan.FromMilliseconds(timeout);
            return httpClient;
        }

        /// <summary>
        /// Generate a http request message.
        /// </summary>
        /// <param name="method">The method.</param>
        /// <param name="uri">The uri.</param>
        /// <param name="headers">The request headers.</param>
        /// <param name="content">The content.</param>
        /// <returns>The HttpRequestMessage.</returns>
        public static async Task<HttpRequestMessage> GenerateRequest(HttpMethod method, string uri, IDictionary<string, string> headers = null, string content = null)
        {
            HttpRequestMessage request = new HttpRequestMessage(method, uri);
            if (headers != null && headers.Count > 0)
            {
                foreach (var header in headers)
                {
                    request.Headers.Add(header.Key.ToLowerInvariant(), header.Value);
                }
            }

            if (content != null)
            {
                request.Content = new StringContent(content, Encoding.UTF8, "application/json");
            }

            return await Task.FromResult(request);
        }

        /// <summary>
        /// Send HTTP request in a reliable way..
        /// </summary>
        /// <param name="httpClient">The HTTP client.</param>
        /// <param name="request">The HTTP request to send.</param>
        /// <param name="errorDetectionStrategy">The HTTP fault detection strategy. True indicates that this HTTP request requires retry.</param>
        /// <param name="onRetrying">The method being invoked before retry.</param>
        /// <param name="maxRetries">The maximum retries.</param>
        /// <param name="retryIntervalInMS">The retry interval in milliseconds.</param>
        /// <param name="cancellationToken">The CancellationToken.</param>
        /// <returns>Returns the HTTP response message.</returns>
        public static async Task<HttpResponseMessage> ReliableSendAsync(
            this HttpClient httpClient,
            Func<Task<HttpRequestMessage>> request,
            Func<HttpResponseMessage, bool> errorDetectionStrategy = null,
            Action<HttpRetryEventArgs> onRetrying = null,
            int maxRetries = DefaultMaximumRetryCount,
            int retryIntervalInMS = DefaultRetryIntervalMilliseconds,
            CancellationToken cancellationToken = new CancellationToken())
        {
            if (request == null)
            {
                throw new ArgumentNullException(nameof(request));
            }

            var currentIteration = 0;
            var lastHttpStatus = HttpStatusCode.OK;
            string lastHttpResponseString = null;
            HttpResponseMessage lastHttpResponse = null;
            while (currentIteration <= maxRetries)
            {
                HttpResponseMessage response = null;
                var requestTime = new Stopwatch();
                using (var httpRequest = await request())
                {
                    try
                    {
                        requestTime.Start();
                        response = await httpClient.SendAsync(httpRequest, cancellationToken);
                        if (response == null || !response.IsSuccessStatusCode || (errorDetectionStrategy != null && errorDetectionStrategy(response) == false))
                        {
                            throw new Exception($"Sending request to feed endpoint {httpRequest?.RequestUri?.ToString()}. StatusCode: {response?.StatusCode}. Content: {response?.Content?.ReadAsStringAsync()?.GetAwaiter().GetResult()}");
                        }

                        lastHttpResponseString = response?.Content?.ReadAsStringAsync()?.GetAwaiter().GetResult();
                    }
                    catch (Exception ex)
                    {
                        lastHttpResponseString = ex.Message;
                    }
                    finally
                    {
                        lastHttpStatus = response?.StatusCode ?? HttpStatusCode.InternalServerError;
                        requestTime.Stop();
                    }
                }

                // Terminating retry when (or): 
                // 1) Reach maximum iteration 
                // 2) Is Canceled
                if (currentIteration++ >= maxRetries || cancellationToken.IsCancellationRequested)
                {
                    lastHttpResponse = response;
                    break;
                }

                if (response != null)
                {
                    // Terminating retry when (or): 
                    // 3) Success code 
                    // 4) Not match detection strategy
                    if (response.IsSuccessStatusCode || (errorDetectionStrategy != null && errorDetectionStrategy(response) == false))
                    {
                        lastHttpResponse = response;
                        break;
                    }

                    response.Dispose();
                }

                await Task.Delay(retryIntervalInMS);
                onRetrying?.Invoke(new HttpRetryEventArgs(
                    currentIteration,
                    TimeSpan.FromMilliseconds(currentIteration * retryIntervalInMS),
                    lastHttpStatus,
                    TimeSpan.FromMilliseconds(requestTime.ElapsedMilliseconds),
                    lastHttpResponseString));
            }

            return lastHttpResponse;
        }
    }
}


#endregion
{% endhighlight %}