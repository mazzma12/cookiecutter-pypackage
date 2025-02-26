.PHONY: clean clean-build clean-pyc clean-test coverage dist docs help install lint lint/ruff release-patch release-minor release-major
        {%- if cookiecutter.use_black == 'y' %} lint/black{% endif %}
.DEFAULT_GOAL := help

define BROWSER_PYSCRIPT
import os, webbrowser, sys

from urllib.request import pathname2url

webbrowser.open("file://" + pathname2url(os.path.abspath(sys.argv[1])))
endef
export BROWSER_PYSCRIPT

define PRINT_HELP_PYSCRIPT
import re, sys

for line in sys.stdin:
	match = re.match(r'^([a-zA-Z_-]+):.*?## (.*)$$', line)
	if match:
		target, help = match.groups()
		print("%-20s %s" % (target, help))
endef
export PRINT_HELP_PYSCRIPT

BROWSER := python -c "$$BROWSER_PYSCRIPT"

BOLD=$$(tput bold)
NORMAL=$$(tput sgr0)

help:
	@python -c "$$PRINT_HELP_PYSCRIPT" < $(MAKEFILE_LIST)

clean: clean-build clean-pyc clean-test ## remove all build, test, coverage and Python artifacts

clean-build: ## remove build artifacts
	rm -fr build/
	rm -fr dist/
	rm -fr .eggs/
	find . -name '*.egg-info' -exec rm -fr {} +
	find . -name '*.egg' -exec rm -f {} +

clean-pyc: ## remove Python file artifacts
	find . -name '*.pyc' -exec rm -f {} +
	find . -name '*.pyo' -exec rm -f {} +
	find . -name '*~' -exec rm -f {} +
	find . -name '__pycache__' -exec rm -fr {} +

clean-test: ## remove test and coverage artifacts
	rm -fr .tox/
	rm -f .coverage
	rm -fr htmlcov/
	rm -fr .pytest_cache

doctest:
	python -m pytest --doctest-modules {{ cookiecutter.project_slug }} tests

{%- if cookiecutter.use_black == 'y' %}
lint/black: ## check style with black
	black --check {{ cookiecutter.project_slug }} tests
{%- endif %}
lint/ruff:
	ruff {{ cookiecutter.project_slug }} tests

lint: lint/ruff{%- if cookiecutter.use_black == 'y' %} lint/black{%- endif %} ## check style

test: ## run tests quickly with the default Python
{%- if cookiecutter.use_pytest == 'y' %}
	python -m pytest
{%- else %}
	python setup.py test
{%- endif %}

test-all: ## run tests on every Python version with tox
	tox

coverage: ## check code coverage quickly with the default Python
{%- if cookiecutter.use_pytest == 'y' %}
	coverage run --source {{ cookiecutter.project_slug }} -m pytest
{%- else %}
	coverage run --source {{ cookiecutter.project_slug }} setup.py test
{%- endif %}
	coverage report -m
	coverage html
	$(BROWSER) htmlcov/index.html

docs: ## generate Sphinx HTML documentation, including API docs
	pip install -r docs/requirements.txt
	rm -f docs/{{ cookiecutter.project_slug }}.rst
	rm -f docs/modules.rst
	sphinx-apidoc -o docs/ {{ cookiecutter.project_slug }}
	$(MAKE) -C docs clean
	$(MAKE) -C docs html
	$(BROWSER) docs/_build/html/index.html

servedocs: docs ## compile the docs watching for changes
	watchmedo shell-command -p '*.rst' -c '$(MAKE) -C docs html' -R -D .

{%- if cookiecutter.pypi_username != "" %}
release: dist ## package and upload a release
	twine upload dist/*
{%- endif %}

release-patch:
	@# Remove the trailing -dev0 from version number and tag the new version
	bumpversion release --tag
	@echo "${BOLD}Tagged version: $$(bumpversion --dry-run --list patch | grep current_version | cut -d= -f2)${NORMAL}"
	@make dev

release-minor:
	@# Increment to the new minor
	bumpversion minor --no-commit
	@# Remove the trailing -dev0 from version number
	bumpversion release --allow-dirty --tag
	@echo "${BOLD}Tagged version: $$(bumpversion --dry-run --list patch | grep current_version | cut -d= -f2)${NORMAL}"
	@make dev

release-major:
	@# Increment to the new major
	bumpversion major --no-commit
	@# Remove the trailing -dev0 from version number
	bumpversion release --allow-dirty --tag
	@echo "${BOLD}Tagged version: $$(bumpversion --dry-run --list patch | grep current_version | cut -d= -f2)${NORMAL}"
	@make dev

dev:
	@# Increment to the new patch (automatically adds -dev0 to it)
	bumpversion patch --message "Post-release: {new_version}"
	@echo "${BOLD}Current version: $$(bumpversion --dry-run --list patch | grep current_version | cut -d= -f2)${NORMAL}"

dist: clean ## builds source and wheel package
	python setup.py sdist
	python setup.py bdist_wheel
	ls -l dist

install: clean ## install the package to the active Python's site-packages
	pip install -e ".[dev,test]"
	{% if cookiecutter.use_pre_commit == 'y' -%}
	pre-commit install{% endif %}
