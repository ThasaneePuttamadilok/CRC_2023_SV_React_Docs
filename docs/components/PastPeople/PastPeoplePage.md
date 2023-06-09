# PastPeoplePage Component
The `PastPeoplePage` component represents a page that displays information about past people. It includes a header, navigation bar, introduction, and buttons for navigating to related sections. The component utilizes the Material-UI library for styling.

## Usage
To use the `PastPeoplePage` component, follow these steps:

- Import the necessary components and styles from the Material-UI library and other files:
```jsx
import { Box, Grid } from '@mui/material';
import HeaderLogoSearch from '../header/HeaderSearchLogo';
import NavBarPeople from './Header/NavBarPeople';
import PersonImage from '@/assets/personImg.png';
import PEOPLE from '@/utils/flatfiles/people_page_data.json';
import '@/style/page-past.scss';
import { Link } from 'react-router-dom';

```
- Define the `PastPeoplePage` component:
```jsx
const PastPeoplePage = () => {
  const currentDate = new Date();
  const currentYear = currentDate.getFullYear();

  return (
    <>
      <HeaderLogoSearch />
      <NavBarPeople />
      <div className="page" id="main-page-past-home">
        <Box
          sx={{
            flexGrow: 1,
            marginTop: {
              sm: '2rem',
              md: '8%',
            },
          }}
        >
          <Grid container spacing={2}>
            <Grid item xs={4} className="grid-people-image">
              <img
                className="flipped-image"
                src={PersonImage}
                alt="PersonImage"
              />
            </Grid>
            <Grid item xs={8} className="grid-people-introduction">
              {PEOPLE.map((item, index) => (
                <div key={index}>
                  <div>{item.text_introuduce}</div>
                  <div>{item.text_description}</div>
                </div>
              ))}
              <div className="btn-Enslaved-enslavers">
                <Link
                  to="/PastHomePage/enslaved"
                  style={{ textDecoration: 'none' }}
                >
                  <div className="enslaved-btn">Enslaved</div>
                </Link>
                <Link
                  to="/PastHomePage/enslaver"
                  style={{ textDecoration: 'none' }}
                >
                  <div className="enslavers-btn">Enslavers</div>
                </Link>
              </div>
            </Grid>
          </Grid>
        </Box>

        <div className="credit-bottom-right">{`Credit: Artist Name ${currentYear}`}</div>
      </div>
    </>
  );
};

```
- Use the `PastPeoplePage` component in your application:
```jsx
<PastPeoplePage />

```
n the above example, the `PastPeoplePage` component will be rendered as a page in your application, displaying information about past people.

Feel free to customize the component's styling and content to fit your specific use case.

## Props
The `PastPeoplePage` component does not accept any props.

## External Dependencies
The `PastPeoplePage` component requires the following external dependencies:

Material-UI `(@mui/material)`
React Router DOM `(react-router-dom)`
Make sure to install and import these dependencies before using the `PastPeoplePage` component.