<?php
class CPayeer
{
	private $url = 'https://payeer.com/ajax/api/api.php';
	private $agent = 'Mozilla/5.0 (Windows NT 6.1; rv:12.0) Gecko/20100101 Firefox/12.0';

	private $auth = array();

	private $output;
	private $errors;
	private $language = 'ru';

	public function __construct($account, $apiId, $apiPass)
	{
		$arr = array(
			'account' => $account,
			'apiId' => $apiId,
			'apiPass' => $apiPass,
		);

		$response = $this->getResponse($arr);

		if ($response['auth_error'] == '0')
		{
			$this->auth = $arr;
		}
	}

	public function isAuth()
	{
		if (!empty($this->auth)) return true;
		return false;
	}

	private function getResponse($arPost)
	{
		if (!function_exists('curl_init'))
		{
		   die('curl library not installed');
		   return false;
		}

		if ($this->isAuth())
		{
			$arPost = array_merge($arPost, $this->auth);
		}

		$data = array();
		foreach ($arPost as $k => $v)
		{
			$data[] = urlencode($k) . '=' . urlencode($v);
		}
		$data[] = 'language=' . $this->language;
		$data = implode('&', $data);

		$handler  = curl_init();
		curl_setopt($handler, CURLOPT_URL, $this->url);
		curl_setopt($handler, CURLOPT_HEADER, 0);
		curl_setopt($handler, CURLOPT_POST, true);
		curl_setopt($handler, CURLOPT_POSTFIELDS, $data);
		curl_setopt($handler, CURLOPT_SSL_VERIFYPEER, 0);
		curl_setopt($handler, CURLOPT_SSL_VERIFYHOST, 0);
		curl_setopt($handler, CURLOPT_USERAGENT, $this->agent);
		curl_setopt($handler, CURLOPT_RETURNTRANSFER, 1);

		$content = curl_exec($handler);
		//print_r($content);

		$arRequest = curl_getinfo($handler);
		//print_r($arRequest);

		curl_close($handler);
		//print_r($content);

		$content = json_decode($content, true);

		if (isset($content['errors']) && !empty($content['errors']))
		{
			$this->errors = $content['errors'];
		}

		return $content;
	}

	public function getPaySystems()
	{
		$arPost = array(
			'action' => 'getPaySystems',
		);

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function initOutput($arr)
	{
		$arPost = $arr;
		$arPost['action'] = 'initOutput';

		$response = $this->getResponse($arPost);

		if (empty($response['errors']))
		{
			$this->output = $arr;
			return true;
		}

		return false;
	}

	public function output()
	{
		$arPost = $this->output;
		$arPost['action'] = 'output';

		$response = $this->getResponse($arPost);

		if (empty($response['errors']))
		{
			return $response['historyId'];
		}

		return false;
	}

	public function getHistoryInfo($historyId)
	{
		$arPost = array(
			'action' => 'historyInfo',
			'historyId' => $historyId
		);

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function getBalance()
	{
		$arPost = array(
			'action' => 'balance',
		);

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function getErrors()
	{
		return $this->errors;
	}

	public function transfer($arPost)
	{
		$arPost['action'] = 'transfer';

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function SetLang($language)
	{
		$this->language = $language;
		return $this;
	}

	public function getShopOrderInfo($arPost)
	{
		$arPost['action'] = 'shopOrderInfo';

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function checkUser($arPost)
	{
		$arPost['action'] = 'checkUser';

		$response = $this->getResponse($arPost);

		if (empty($response['errors']))
		{
			return true;
		}

		return false;
	}

	public function getExchangeRate($arPost)
	{
		$arPost['action'] = 'getExchangeRate';

		$response = $this->getResponse($arPost);

		return $response;
	}

	public function merchant($arPost)
	{
		$arPost['action'] = 'merchant';

		$arPost['shop'] = json_encode($arPost['shop']);
		$arPost['form'] = json_encode($arPost['form']);
		$arPost['ps'] = json_encode($arPost['ps']);

		if (empty($arPost['ip'])) $arPost['ip'] = $_SERVER['REMOTE_ADDR'];

		$response = $this->getResponse($arPost);

		if (empty($response['errors']))
		{
			return $response;
		}

		return false;
	}
}
?>--
layout: default
---

# Contributing to Open Source Guides

Thanks for checking out the Open Source Guides! We're excited to hear and learn from you. Your experiences will benefit others who read and use these guides.

We've put together the following guidelines to help you figure out where you can best be helpful.

## Table of Contents

0. [Types of contributions we're looking for](#types-of-contributions-were-looking-for)
0. [Ground rules & expectations](#ground-rules--expectations)
0. [How to contribute](#how-to-contribute)
0. [Style guide](#style-guide)
0. [Setting up your environment](#setting-up-your-environment)
0. [Community](#community)

## Types of contributions we're looking for
There are many ways you can directly contribute to the guides (in descending order of need):

* Fix editorial inconsistencies or inaccuracies
* Add stories, examples, or anecdotes that help illustrate a point
* Revise language to be more approachable and friendly
* [Translate guides into other languages](docs/translations.md)

Interested in making a contribution? Read on!

## Ground rules & expectations

Before we get started, here are a few things we expect from you (and that you should expect from others):

* Be kind and thoughtful in your conversations around this project. We all come from different backgrounds and projects, which means we likely have different perspectives on "how open source is done." Try to listen to others rather than convince them that your way is correct.
* Open Source Guides are released with a [Contributor Code of Conduct](./CODE_OF_CONDUCT.md). By participating in this project, you agree to abide by its terms.
* If you open a pull request, please ensure that your contribution passes all tests. If there are test failures, you will need to address them before we can merge your contribution.
* When adding content, please consider if it is widely valuable. Please don't add references or links to things you or your employer have created as others will do so if they appreciate it.

## How to contribute

If you'd like to contribute, start by searching through the [issues](https://github.com/github/opensource.guide/issues) and [pull requests](https://github.com/github/opensource.guide/pulls) to see whether someone else has raised a similar idea or question.

If you don't see your idea listed, and you think it fits into the goals of this guide, do one of the following:
* **If your contribution is minor,** such as a typo fix, open a pull request.
* **If your contribution is major,** such as a new guide, start by opening an issue first. That way, other people can weigh in on the discussion before you do any work.

## Style guide
If you're writing content, see the [style guide](./docs/styleguide.md) to help your prose match the rest of the Guides.

## Setting up your environment

This site is powered by [Jekyll](https://jekyllrb.com/). Running it on your local machine requires a working [Ruby](https://www.ruby-lang.org/en/) installation with [Bundler](https://bundler.io/).

Once you have that set up, run:

    script/bootstrap
    script/server

…and open http://localhost:4000 in your web browser.

## Community

Discussions about the Open Source Guides take place on this repository's [Issues](https://github.com/github/opensource.guide/issues) and [Pull Requests](https://github.com/github/opensource.guide/pulls) sections. Anybody is welcome to join these conversations. There is also a [mailing list](http://eepurl.com/cecpnT) for regular updates.

Wherever possible, do not take these conversations to private channels, including contacting the maintainers directly. Keeping communication public means everybody can benefit and learn from the conversation.
